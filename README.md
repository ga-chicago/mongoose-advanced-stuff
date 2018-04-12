# More Advanced Mongoose/Modeling techniques


## Extend the abilities of models

You can give all models using a specific schema extra properties and methods when creating that schema

```javascript
//after initial schema declaration
articleSchema.methods.longTitle = ()=>{
	return this.author + ": " + this.title;
}

//instantiate a model and then call it
const article = new Article();
console.log(article.longTitle());
```

## Extend the abilities of classes

You can create static methods for model constructor functions in their schema

```javascript

//after initial schema declaration
articleSchema.statics.search = function(name, cb){ //because we use this here, we'll need an old-fashioned function
	//In this situation, it's like .find(), but it searches both title and author
	return this.find({
		$or : [
			{ title: new RegExp(name, 'i') },
			{ author: new RegExp(name, 'i') }
		]
	}, cb);
}

//call the static method
Article.search('Some', (err, data) => {
	console.log(data);
});
```

## Reference other models by id

Sub docs are difficult when it comes to updating, because all duplicates must be updated.  But we can reference by ID to get around this.

### Single ObjectId reference

```javascript
const mongoose = require('mongoose');
const Schema = mongoose.Schema;
mongoose.connect('mongodb://localhost:27017/test');

const articleSchema = new Schema({
	title: { type: String },
	author: { type: Schema.Types.ObjectId, ref: 'Author' } //the author property is just an id of another object
});

const authorSchema = new Schema({
	name: { type: String }
});

const Article = mongoose.model('Article', articleSchema);
const Author = mongoose.model('Author', authorSchema);

const matt = new Author({name: 'Matt'});
matt.save(()=>{
	const article1 = new Article({title:'Awesome Title', author: matt._id});
	article1.save(()=>{
		showAll();
	});
});

const showAll = ()=>{
	Article.find().populate('author').exec((error, article)=>{ //dynamically switch out any ids with the objects they reference
		console.log(article);
		mongoose.connection.close();
	})
};
```

### Arrays of ObjectId references

We can do the same with arrays of ids

```javascript
const authorSchema = new Schema({
	name: { type: String },
	articles: [{type: Schema.Types.ObjectId, ref: 'Articles'}]
});

const Article = mongoose.model('Article', articleSchema);
const Author = mongoose.model('Author', authorSchema);

const matt = new Author({name: 'Matt'});
let article_id;
matt.save(()=>{
	let article1 = new Article({title:'Awesome Title', author: matt._id});
	article1.save(()=>{
		article_id = article1._id;
		matt.articles.push(article1);
		matt.save(showAll);
	});
});

var showAll = (err, author)=>{
	Author.find().populate('articles').exec((err, authors)=>{ //dynamically switch out any ids with the objects they reference
		console.log(authors);
		mongoose.connection.close();
	});
};
```

## Create actions for models to execute during database actions

We can perform actions before and after database work

```javascript
articleSchema.pre('save', function(next){ //because we use this here, we'll need an old-fashioned function
	console.log(this);
	console.log('saving to backup database');
	next();
});
```
async
```javascript
articleSchema.pre('save', true, function(next){ //because we use this here, we'll need an old-fashioned function
	console.log(this);
	console.log('saving to backup database');
	next();
	doAsync(done);
});
```
post
```javascript
articleSchema.post('save', function(next){ //because we use this here, we'll need an old-fashioned function
	console.log('saving complete');
});
