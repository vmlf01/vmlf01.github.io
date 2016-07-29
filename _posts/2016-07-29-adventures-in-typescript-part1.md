---
layout: post
title: "Adventures in Typescript, part 1"
description:
headline:
modified: 2016-07-29 09:05:14 +0100
category: [typescript, webdevelopment, js, code, nodejs]
tags: [typescript, nodejs, js]
imagefeature: adventures-typescript.jpg
mathjax:
chart:
comments: true
featured: false
---

ES2015 brings with it the ability to have modules and specifies several variations of import/export statements to use them. Typescript 1.5 changed its [module definition](https://www.typescriptlang.org/docs/handbook/modules.html) to realign with ES2015 specification, so it supports the same syntax.

When transpiling Node.js code to ES5, Typescript will transform the import/export statements into the CommonJS syntax that is used in Node.js.

Say you got some file like:

```js
// calc.test.ts
import 'should';
import calc from '../src/calc';

describe('calc', () => {
    it('should  calculate 1+1', () => {
        calc.add(1, 1).should.equal(2);
    });
});
```

This will transpile into:

```js
// calc.test.js
"use strict";
require('should');
var calc_1 = require('../src/calc');
describe('calc', function () {
    it('should  calculate 1+1', function () {
        calc_1.default.add(1, 1).should.equal(2);
    });
});
```

Seems ok, you get two require statements with the 'should' and 'calc' modules. However, sometimes you need a reference to 'should' to be able to perform ```should(...)``` style assertions.
No problem, you can do:

```js
// calc.test.ts
import * as should from 'should';
import calc from '../src/calc';

describe('calc', () => {
    it('should  calculate 1+1', () => {
        const result = calc.add(1, 1);
        should.exist(result);
        result.should.equal(2);
    });
});
```

and you get:

```js
// calc.test.js
"use strict";
var should = require('should');
var calc_1 = require('../src/calc');
describe('calc', function () {
    it('should  calculate 1+1', function () {
        var result = calc_1.default.add(1, 1);
        should.exist(result);
        result.should.equal(2);
    });
});
```

which is ok also. The problem comes when you want to be consistent and always use this last syntax, even if you don't call ```should(...)```

```js
// calc.test.ts
import * as should from 'should';
import calc from '../src/calc';

describe('calc', () => {
    it('should  calculate 1+1', () => {
        const result = calc.add(1, 1);
        result.should.equal(2);
    });
});
```

```js
// calc.test.js
"use strict";
var calc_1 = require('../src/calc');
describe('calc', function () {
    it('should  calculate 1+1', function () {
        var result = calc_1.default.add(1, 1);
        result.should.equal(2);
    });
});
```

Notice anything missing? Where's the ```require('should')``` gone to? Turns out Typescript tries to be smart and [removes the un-used import statements](https://github.com/Microsoft/TypeScript/issues/4717).
Problem is, taking 'should' as an example, there are some modules that do stuff just by requiring them. In this case, 'should' adds a ```should()``` method to the Object prototype so you can chain it in your assertions.
Typescript has no way of knowing this, so it thinks 'should' is not used anywhere and doesn't generate the corresponding require statement.
If you want to always use the same syntax without worrying about whether the reference is used or not, the solution is to declare a local variable and set it to the import reference.

```js
// calc.test.ts
import * as _should from 'should';
import calc from '../src/calc';

const should = _should;
```

This way, you ensure that the ```_should``` reference is used and Typescript will no longer remove the 'should' import from transpiled code.
