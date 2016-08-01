---
layout: post
title: "Adventures in Typescript, part 2"
description: A look at how to use Mongoose with Typescript, specially how to define Model static methods 
headline:
modified: 2016-08-01 10:28:39 +0100
category: [typescript, mongodb, js, code, nodejs]
tags: [typescript, mongodb, nodejs, js]
imagefeature: Black_mongoose_waterberg.jpg
mathjax:
chart:
comments: true
featured: false
---

Suppose you got a MEAN stack API project using Mongoose for data access. Now imagine you got it using Typescript.
It doesn't make that much difference in terms of project structure, you just got to install the typings for MongoDB and Mongoose and you're set to go, right?

```sh
$ typings install --save --global --source env node
$ typings install --save --global --source dt mongodb
$ typings install --save --global --source dt mongoose
$ typings install --save --global --source dt es6-promise
$ typings install --save --global --source dt mongoose-promise
```

### Adding types to our models

Well, you need one more step, which is to provide your own types information. So let's define an interface for our model:

```js
// IPerson.ts
interface IPerson {
    name: string;
}

export default IPerson
```

```js
// IPersonDocument.ts
import * as mongoose from 'mongoose'

import IPerson from './IPerson'

interface IPersonDocument extends IPerson, mongoose.Document {
}

export default IPersonDocument
```

This [styleguide](https://github.com/Appsilon/styleguide/wiki/mongoose-typescript-models) I found on GitHub explains in more detail the reasons for creating it in two steps, but it basically boils down to decoupling the model from mongoose details.

Next, we define our mongoose Person model:

```js
// PersonModel.ts
import * as mongoose from 'mongoose';

import IPersonDocument from './IPersonDocument';

const personSchema = new mongoose.Schema({
    name: {
        type: mongoose.SchemaTypes.String,
        required: true
    }
});

const PersonModel: mongoose.Model<IPersonDocument> = 
    mongoose.model<IPersonDocument>('Person', personSchema);

export default PersonModel;
```

Now you can use it where you need to:

```js
// PersonService.ts
import * as mongoose from 'mongoose';

import IPerson from './IPerson';
import IPersonDocument from './IPersonDocument';

class PersonService {
    constructor(private PersonModel: mongoose.Model<IPersonDocument>) {
    }

    getAll(): Promise<IPerson[]> {
        return this.PersonModel.find();
    }
}

export default PersonService;
```

And we got a PersonService we can use to fetch data from the database. But you don't want to fetch all items at once, suppose you got millions of them?

### Using model static methods

This is the kind of thing you will need for most models, and there are already a few npm modules dedicated to solving this problem.
Let's use [mongoose-paginate](https://www.npmjs.com/package/mongoose-paginate) and following the instructions, we change out schema to look like this:

```js
// PersonModel.ts
import * as mongoose from 'mongoose';
import * as mongoosePaginate from 'mongoose-paginate';

import IPersonDocument from './IPersonDocument';

const personSchema = new mongoose.Schema({
    name: {
        type: mongoose.SchemaTypes.String,
        required: true
    }
});
schema.plugin(mongoosePaginate);

const PersonModel: mongoose.Model<IPersonDocument> = 
    mongoose.model<IPersonDocument>('Person', personSchema);

export default PersonModel;
```

So we add a plugin to our person schema, which will add a new static ```Model.paginate()``` method to the model we can use instead of ```Model.find()```.
Straight away we see a problem, there isn't a type definitions file for mongoose-paginate! Fortunately, we can use a skeleton from a [GitHub issue](https://github.com/edwardhotchkiss/mongoose-paginate/issues/82) to get us started for now:

```js
// mongoose-paginate.d.ts
declare module 'mongoose-paginate' {
  import * as mongoose from 'mongoose';

  namespace mongoosePaginate {
    interface IOptions {
      select?: Object | string;
      sort?: Object | string;
      populate?: Array<string> | Object | string;
      lean?: boolean;
      leanWithId?: boolean;
      offset?: number;
      page?: number;
      limit?: number;
    }

    interface IMongoosePaginate {
      paginate(query?: Object, options?: IOptions, callback?: (err: Object, result: any) => any): any;
    }

    interface IMongoosePaginatePlugin extends IMongoosePaginate {
      (schema: mongoose.Schema): any;
    }
  }

  const mongoosePaginate: mongoosePaginate.IMongoosePaginatePlugin;
  export = mongoosePaginate;
}
```

The service code now looks like:

```js
    // ...
    getAll(page: number): Promise<IPerson[]> {
        return this.PersonModel.paginate({}, { page: page, limit: 10 });
    }
    // ...
```

This gives another problem, because Typescript has no way of knowing we added a static method to the PersonModel.

```sh
$ ./node_modules/.bin/tsc
PersonService.ts(12,29): error TS2339: Property 'paginate' does not exist on type 'IModelConstructor<IPersonDocument> & EventEmitter'.
```

There's a lot of types in TypeScript!

### Extending our model type definition

The first approach that comes to mind is just adding the mongoose-paginate interface to the list of interfaces our model inherits from:

```js
// PersonModel.ts
import * as mongoose from 'mongoose';
import * as mongoosePaginate from 'mongoose-paginate';

import IPersonDocument from './IPersonDocument';

const personSchema = new mongoose.Schema({
    name: {
        type: mongoose.SchemaTypes.String,
        required: true
    }
});
schema.plugin(mongoosePaginate);

export interface IPersonModel extends  mongoose.Model<IPersonDocument>, mongoosePaginate.IMongoosePaginate {
}

const PersonModel: IPersonModel = 
    mongoose.model<IPersonDocument>('Person', personSchema) as IPersonModel;

export default PersonModel;
```

I saw this in a [GitHub issue](https://github.com/vagarenko/mongoose-typescript-definitions/issues/4) but that doesn't work, you'll get errors like these:

```sh
PersonModel.ts(14,39): error TS2312: An interface may only extend a class or another interface.
PersonModel.ts(18,3): error TS2352: Neither type 'IModelConstructor<IPersonDocument> & EventEmitter' nor type 'IPersonModel' is assignable to the other.
  Type 'EventEmitter' is not assignable to type 'IPersonModel'.
    Property 'paginate' is missing in type 'EventEmitter'.
```

Might be due to changes in the mongoose definitions file, because mongoose.Model<T> is defined as a Type, not a class or interface, so you cannot extend from it.

### Typescript advanced types to the rescue

Typescript has several [advanced type features](https://www.typescriptlang.org/docs/handbook/advanced-types.html), including Intersection Types and Type Aliases.
Together, they help us solve this problem. Let's define a new Type alias that combines the types we want:

```js
// TPaginatedModel.ts
import * as mongoose from 'mongoose';
import * as mongoosePaginate from 'mongoose-paginate';

import IPersonDocument from './IPersonDocument';

type TPaginatedModel<T extends mongoose.Document> = mongoose.Model<T> & mongoosePaginate.IMongoosePaginate;

export default TPaginatedModel;
```

This defines a new type that is both a mongoose.Model<T>  and an IMongoosePaginate, so it will merge the type definitions from both to create a new TPaginatedModel<T> type.

With this new type, we can now define our model as:

```js
// PersonModel.ts
import * as mongoose from 'mongoose';
import * as mongoosePaginate from 'mongoose-paginate';

import TPaginatedModel from './TPaginatedModel';
import IPersonDocument from './IPersonDocument';

const personSchema = new mongoose.Schema({
  name: {
    type: mongoose.SchemaTypes.String,
    required: true
  }
});
personSchema.plugin(mongoosePaginate);

const PersonModel: TPaginatedModel<IPersonDocument> = 
    mongoose.model<IPersonDocument>('Person', personSchema) as TPaginatedModel<IPersonDocument>;

export default PersonModel;
```

And our service also ends up with no references to Mongoose:

```js
import IPerson from './IPerson';
import IPersonDocument from './IPersonDocument';
import TPaginatedModel from './TPaginatedModel';

class PersonService {
  constructor(private PersonModel: TPaginatedModel<IPersonDocument>) {
  }

  getAll(page: number): Promise<IPerson[]> {
    return this.PersonModel.paginate({}, { page: page, limit: 10 });
  }
}

export default PersonService
```

Finally, mission accomplished, we get zero errors when compiling our Typescript code and we get IntelliSense for our models!


###### Cover image: Sfbergo - [Black mongoose photographed at Waterberg Plateau, Namibia](https://en.wikipedia.org/wiki/Mongoose#/media/File:Black_mongoose_waterberg.jpg) - CC BY-SA 3.0



