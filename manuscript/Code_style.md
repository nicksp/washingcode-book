{#code-style}

# Code style

I used to be very strict about [code style](https://blog.sapegin.me/all/prettier/). I thought my code style was better than others’, but later I’ve realized that it was just different. And it wasn’t the most popular, so anyone else’s code looked wrong to me.

For example, after reading the [The Programmers’ Stone](https://www.datapacrat.com/Opinion/Reciprocality/r0/index.html) I was formatting braces like this for a long time:

<!-- const food = 'pizza', alert = () => {} -->

<!-- prettier-ignore -->
```js
if (food === 'pizza')
{
  alert('Pizza ;-)');
}
else
{
  alert('Not pizza ;-(');
}
```

Or I had two spaces in front of inline comments to better separate them from code:

<!-- prettier-ignore -->
```js
const volume = 200;  // ml
```

So if any other developer touched my code, they would definitely make it inconsistent, because unlikely they would follow _my code style_ — so uncommon it was. And code review would be a nightmare if I wanted to enforce _my code style_.

## Not all code styles are good

I wasn’t entirely wrong though: not every code style makes code easy to read and maintain.

For example, this way of defining arrays makes it harder to move or add new items:

<!-- prettier-ignore -->
```js
const dogs = [
  'dachshund',
  'saluki',
  'sheltie'
];
```

That’s because we’ll have to change two lines every time we want to do something at the array’s end. It also clutters the diff for the same reason:

```diff
const dogs = [
  'dachshund',
  'saluki',
-  'sheltie'
+  'sheltie',
+  'whippet'
];
```

[Trailing, or dangling, commas](https://medium.com/@nikgraf/why-you-should-enforce-dangling-commas-for-multiline-statements-d034c98e36f8) solve both problems without making code any harder to write or read:

<!-- prettier-ignore -->
```js
const dogs = [
  'dachshund',
  'saluki',
  'sheltie',
];
```

Now we need to change only one line:

```diff
const dogs = [
  'dachshund',
  'saluki',
  'sheltie',
+  'whippet',
];
```

And if there was one particular code style I really dislike, it would be code blocks without brackets:

<!-- function test(pizza) { -->

```js
if (pizza) pizza();
```

<!--
}
let test2 = false;
test()
expect(test2).toBe(false);
test(() => {test2 = true})
expect(test2).toBe(true);
-->

Instead of:

<!-- function test(pizza) { -->

```js
if (pizza) {
  pizza();
}
```

<!--
}
let test2 = false;
test()
expect(test2).toBe(false);
test(() => {test2 = true})
expect(test2).toBe(true);
-->

I think it really damages readability of the code. The longer the condition, the harder it is to see the body:

<!-- function test(recipeDetails) { -->

```js
if (!recipeDetails?.allIngredients.length) return null;
```

<!--
}
expect(test({allIngredients: []})).toBe(null);
expect(test({allIngredients: [1]})).toBe(undefined);
-->

I’m 102% sure, I’d not notice the `return` here when reading the code for the first time.

Compare it with:

<!-- function test(recipeDetails) { -->

```js
if (recipeDetails?.allIngredients.length === 0) {
  return null;
}
```

<!--
}
expect(test({allIngredients: []})).toBe(null);
expect(test({allIngredients: [1]})).toBe(undefined);
-->

Now the `return` statement is more noticeable.

Of course, there are worse ways of writing code:

<!-- function test(pizza) { -->

```js
pizza && pizza();
```

<!--
}
let test2 = false;
test()
expect(test2).toBe(false);
test(() => {test2 = true})
expect(test2).toBe(true);
-->

Shorter isn’t always better. (I’m fine with `pizza?.()` though, sometimes.)

**Tip:** Use `curly` ESLint rule to make sure all conditions have braces. See more in the [Lint your code](#linting) chapter.

## Obsolete code styles

Sometimes developers follow a particular code style even if the initial reasoning behind it is no longer relevant.

For example, using leading commas in arrays and objects when JavaScript didn’t support trailing commas:

<!-- prettier-ignore -->
```js
const dogs = [
  'dachshund'
, 'saluki'
, 'sheltie'
];
```

The goal of this style was the same as of trailing commas in the previous section — to make adding new items easier and diffs more readable, but there are no reasons to use this anymore: Internet Explorer 8 was the last browser that didn’t support trailing commas.

Another example is [Yoda conditions](https://en.wikipedia.org/wiki/Yoda_conditions), a style where a literal is on the left side of a condition:

<!--
const meaning = 42
let result = false;
-->

<!-- prettier-ignore -->
```js
if (42 === meaning) {
```

<!--
  result = true
}
expect(result).toBe(true)
-->

It’s too easy to type `=` instead of `==` in languages like C and make an assignment:

<!--
let meaning = 0
let result = false;
-->

```js
if (meaning = 42) {
```

<!--
  result = true
}
expect(meaning).toBe(42)
-->

Instead of a comparison:

<!--
const meaning = 42
let result = false;
-->

```js
if (meaning == 42) {
```

<!--
  result = true
}
expect(result).toBe(true)
-->

This is much less relevant in JavaScript where the strict equality (`===`, values and types must be equal) is the preferred style and on most projects a linter will complain if we try to use the loose equality (`==`, only values must be equal). It’s really hard to miss two equal signs when typing `===`. So normal order of conditions is fine and easier to read:

<!--
const meaning = 42
let result = false;
-->

```js
if (meaning === 42) {
```

<!--
  result = true
}
expect(result).toBe(true)
-->

## Nonsensical code styles

Some code styles don’t solve any particular problem, but are expensive to maintain.

For example, aligning object values or right-hands of assignments horizontally to make them look “pretty”:

<!-- prettier-ignore -->
```js
var fs       = require('fs')
  , reamde   = require('reamde')
  , examples = reamde(fs.readFileSync('./README.md', 'utf-8'))
  ;
```

<!-- expect(examples).toEqual('./README.md') -->

Editing code written in such style takes enormous amount of work, and luckily code formatters will remove all the artisanal handcrafted spaces and make code look equally good without requiring any work from a developer:

```js
var fs = require('fs'),
  reamde = require('reamde'),
  examples = reamde(fs.readFileSync('./README.md', 'utf-8'));
```

<!-- expect(examples).toEqual('./README.md') -->

I’d go one step further, and replace a single `var` with a one `var` per line (or `const`):

```js
const fs = require('fs');
const reamde = require('reamde');
const examples = reamde(fs.readFileSync('./README.md', 'utf-8'));
```

<!-- expect(examples).toEqual('./README.md') -->

This will not only make it slightly more readable but also easier to add, remove or move variable declarations.

## Improving readability

In many cases, some ways of writing code are more readable than other ways. Conditions, and especially conditions with negations are good examples. I used to write these as short as possible, now I prefer to be verbose and explicit. Saving a few keystrokes isn’t worth it when the code could be easily misinterpreted. Better learn touch typing.

Consider this example:

<!--
const object = { o: 0 }
let result = false
-->

```js
if (!isEmpty(object)) {
```

<!--
  result = true
}
expect(result).toBe(true)
-->

It’s hard to notice the negation in `!i`. We could rewrite this code to avoid misunderstanding:

<!--
const object = { o: 0 }
let result = false
-->

```js
if (isEmpty(object) === false) {
```

<!--
  result = true
}
expect(result).toBe(true)
-->

Someone may argue that it doesn’t read like English — “not is empty” — but there’s no way someone might miss the negation.

Here’s another example:

<!--
const guacamole = {}
let result = false
-->

```js
if (!('garlic' in guacamole)) {
```

<!--
  result = true
}
expect(result).toBe(true)
-->

This pattern was always awkward to write and read for me until my friend Oleg [opened a whole new world for me](https://twitter.com/oleg008/status/1519593163803049984). We could do the same trick as above to make it more readable:

<!--
const guacamole = {}
let result = false
-->

```js
if ('garlic' in guacamole === false) {
```

<!--
  result = true
}
expect(result).toBe(true)
-->

Another area where condition expansion improves readability is checking array length.

Consider these two examples:

<!--
const puppies = ['Dessi']
let result = false
-->

```js
if (puppies.length) {
```

<!--
  result = true
}
expect(result).toBe(true)
-->

<!--
const puppies = ['Dessi']
let result = false
-->

```js
if (!puppies.length) {
```

<!--
  result = true
}
expect(result).toBe(false)
-->

They look almost the same, and the `!` in front of the second one is easy to miss. Let’s expand them:

<!--
const puppies = ['Dessi']
let result = false
-->

```js
if (puppies.length > 0) {
```

<!--
  result = true
}
expect(result).toBe(true)
-->

<!--
const puppies = ['Dessi']
let result = false
-->

```js
if (puppies.length === 0) {
```

<!--
  result = true
}
expect(result).toBe(false)
-->

Now the conditions look significantly different and there’s no way to misinterpret them.

I’m starting to think that using `!` in conditions is [generally an antipattern](https://twitter.com/Jack_Franklin/status/1189477268764188672), so instead of:

<!--
const isFriday = false
let result = false
-->

```js
if (!isFriday) {
```

<!--
  result = true
}
expect(result).toBe(true)
-->

We should always write:

<!--
const isFriday = false
let result = false
-->

```js
if (isFriday === false) {
```

<!--
  result = true
}
expect(result).toBe(true)
-->

Another example where we can improve readability a bit is checking whether a value is between two numbers. A naïve way to so would like so:

<!-- let inside = false, x = 7 -->

```js
if (x > 3 && x < 13) {
```

<!--
inside = true
}
expect(inside).toBe(true)
-->

This reads like “x is greater than 3 _and_ lower than 13”, which is fine but we can do better:

<!-- let inside = false, x = 7 -->

```js
if (3 < x && x < 13) {
```

<!--
inside = true
}
expect(inside).toBe(true)
-->

Now, it’s easier to see that we want `x` to be _between_ 3 and 13.

We can use the same approach to check whether a value is outside a range:

<!-- let inside = false, x = 7 -->

```js
if (x < 3 || 13 < x) {
```

<!--
inside = true
}
expect(inside).toBe(false)
-->

## Christmas trees vs. sausages

I used to have a limit of 120 characters per line because why not, I have a big screen that could fit a lot of code, I should use the space available to me! Then I lowered it to 100 characters, and then to 80, which is a Prettier’s default value that in their docs they [highly recommend not to change](https://prettier.io/docs/en/options.html#print-width).

An argument for using 80 characters I’ve heard many times but never believed in is that some people may edit code in a terminal that only displays 80 characters, like in the old days. But how many people are really doing this now?

![QuckBasic](images/quickbasic.png)

Some developers seem to prefer long strings, some short.

The longstringers write code that looks like a bunch of sausages on a grill:

<!-- const puppies = [{id: 1, name: 'Dessi', parentId: 3 },{id: 2, name: 'Tsiri', parentId: 3 },{id: 3, name: 'Cthulhu' },] -->

<!-- prettier-ignore -->
```js
const puppiesByParent = {}
puppies.forEach(puppy => {
  if (puppy.parentId) {
    const currentParent = puppies.find(currentPuppy => currentPuppy.id === puppy.parentId)
    puppiesByParent[currentParent.id] = [...(puppiesByParent[currentParent.id] || []), puppy.id]
  }
})
```

<!-- expect(puppiesByParent).toEqual({3: [1, 2]}) -->

Or:

<!--
const origin = 'https://example.com', intl = {formatMessage: () => 'test'}
function test(hasTranslation) {
-->

<!-- prettier-ignore -->
```js
const downloadLink = hasTranslation ? `${origin}/${intl.formatMessage({id: 'download-link'})}` : '/download';
```

<!--
  return downloadLink
}
expect(test(true)).toEqual('https://example.com/test')
expect(test(false)).toEqual('/download')
-->

Or:

<!--
const DACHSHUND_FAVORITE_SNACK = 'yogurt'
function test(breed) {
-->

<!-- prettier-ignore -->
```js
if (breed === 'dachshund') return DACHSHUND_FAVORITE_SNACK;
```

<!--
}
expect(test('dachshund')).toEqual('yogurt')
expect(test('saluki')).toEqual(undefined)
-->

In longstringer code, we’ll see many ternaries, complex string templates, nesting, multiple operations on the same line, long variable names even in a very small scope, an so on.

On the other hand, the shortstringers write code that looks like one side of a Christmas tree:

<!-- const puppies = [{id: 1, name: 'Dessi', parentId: 3 },{id: 2, name: 'Tsiri', parentId: 3 },{id: 3, name: 'Cthulhu' },] -->

```js
const puppiesByParent = puppies.reduce((acc, puppy) => {
  if (puppy.parentId) {
    const parent = puppies.find(x => x.id === puppy.parentId);
    if (parent.id in acc === false) {
      acc[parent.id] = [];
    }
    acc[parent.id].push(puppy.id);
  }
  return acc;
}, {});
```

<!-- expect(puppiesByParent).toEqual({3: [1, 2]}) -->

Or:

<!-- const origin = 'https://example.com', intl = {formatMessage: () => 'test'} -->

```js
function getDownloadLink(hasTranslation) {
  if (hasTranslation) {
    const link = intl.formatMessage({ id: 'download-link' });
    return [origin, link].join('/');
  } else {
    return '/download';
  }
}
```

<!--
expect(getDownloadLink(true)).toEqual('https://example.com/test')
expect(getDownloadLink(false)).toEqual('/download')
-->

Or:

<!--
function test(breed) {
-->

```js
const FAVORITE_SNACKS = {
  dachshund: 'yogurt'
};
if (breed in FAVORITE_SNACKS) {
  return FAVORITE_SNACKS[breed];
}
```

<!--
}
expect(test('dachshund')).toEqual('yogurt')
expect(test('saluki')).toEqual(undefined)
-->

In shortstringer code, we’ll see early returns, extra functions and variables to reduce the number of operations in one line and give things names, less nesting, variable name lengths depending on the size or their scope (longer names for larger scope), and so on.

It’s easier to follow conditions, notice `return` statements in the functions, and generally see what’s happening. The important code — like adding new values to an object — isn’t buried somewhere in a very long line, and clearly separated from the data management code (see [Separate what and how](#separate-what-and-how) chapter). The code has more whitespace and shape that makes it easier to scan.

I’m clearly a shortstringer.

Another issue of the longstringer approach is that Prettier with default setting of 80 characters will likely make the code ugly and quite unreadable:

<!-- const puppies = [{id: 1, name: 'Dessi', parentId: 3 },{id: 2, name: 'Tsiri', parentId: 3 },{id: 3, name: 'Cthulhu' },] -->

```js
const puppiesByParent = {};
puppies.forEach(puppy => {
  if (puppy.parentId) {
    const currentParent = puppies.find(
      currentPuppy => currentPuppy.id === puppy.parentId
    );
    puppiesByParent[currentParent.id] = [
      ...(puppiesByParent[currentParent.id] || []),
      puppy.id
    ];
  }
});
```

<!-- expect(puppiesByParent).toEqual({3: [1, 2]}) -->

Or:

<!--
const origin = 'https://example.com', intl = {formatMessage: () => 'test'}
function test(hasTranslation) {
-->

```js
const downloadLink = hasTranslation
  ? `${origin}/${intl.formatMessage({ id: 'download-link' })}`
  : '/download';
```

<!--
  return downloadLink
}
expect(test(true)).toEqual('https://example.com/test')
expect(test(false)).toEqual('/download')
-->

Or:

<!--
const DACHSHUND_FAVORITE_SNACK = 'yogurt'
function test(breed) {
-->

```js
if (breed === 'dachshund') return DACHSHUND_FAVORITE_SNACK;
```

<!--
}
expect(test('dachshund')).toEqual('yogurt')
expect(test('saluki')).toEqual(undefined)
-->

The shortstringer code stays the same.

And coming back to screens with limited number of characters, I now use 27" screen, split into two windows side by side. Each side fit around 80 characters plus a sidebar with the repository tree.

![Coding on 27" screen](images/27inches.png)

## Make it easy to remember and use

Some conventions are easy to use, some are not so much. Let’s compare three popular conventions for _title casing_:

- **sentence case:** only the first word is capitalized, like in a regular sentence (example: _Breakfast: The most important book about the best meal of the day_);
- **upper case:** all words are capitalized (example: _Breakfast: The Most Important Book About the Best Meal Of The Day_);
- **AP/APA:** see below (example: _Breakfast: The Most Important Book About the Best Meal of the Day_).

The first two are easy to remember: it’s all or nothing. The last one, hovewer, not at all. Here are the rules of the AP/APA title style (quoted from the [ap-style-title-case](https://github.com/words/ap-style-title-case) package docs):

- always capitalize the first word, even if it’s a stop word
- always capitalize the last word, even if it’s a stop word
- lowercase these words: `a`, `an`, `and`, `at`, `but`, `by`, `for`, `in`, `nor`, `of`, `on`, `or`, `so`, `the`, `to`, `up`, `yet`

> Many writers make the error of leaving `to be` verbs in lower case. Even though `is`, `are`, `was`, and `be`, are all short words, they should still be capitalized in a title because they are verbs.
>
> When you write titles that contain prepositions, your word processor will likely tell you that you should leave words like `with`, `about`, and `around` lowercase. Defiantly look past the squiggly line indicating a potential error, and remember that in AP title case, prepositions with four or more letters should be capitalized.

This is way to much to rember and it has too many exceptions to be practical. It also doesn’t make text more readable, and, to my taste, prettier. Automation could help with managing complexity, but then the convention should have significant benefits, and this one doesn’t. That’s why I use sentence case in all my writing, including this book.

This is an extreme case of an inconvenient convention. Programmers rarely go that far but sometimes they try.

One example is [Microsoft .NET naming conventions](https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/capitalization-conventions), where they make a special case for two-letter acronyms:

> The PascalCasing convention, used for all identifiers except parameter names, capitalizes the first character of each word (including acronyms over two letters in length), as shown in the following examples: `PropertyDescriptor`, `HtmlTag`.

> A special case is made for two-letter acronyms in which both letters are capitalized, as shown in the following identifier: `IOStream`.

I don’t see how this improves anything: `IoStream` is easier to read than `IOStream`, and there’s no need to remember about a special rule.

And then even Microsoft couldn’t follow their own guidelines with [XMLHttpRequest](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest).

So, choose the simplest convention, unless more complex rules bring huge benefits. If that is so, thorougly document and automate it.

## The rest doesn’t matter

There are so many ways to write code. For example, we could use function arguments like this:

```js
function ingredientToString(options) {
  return `${options.name} (${options.quantity})`;
}
```

<!-- expect(ingredientToString({name: 'Pizza', quantity: 6})).toBe('Pizza (6)') -->

Or like this:

```js
function ingredientToString(options) {
  const { name, quantity } = options;
  return `${name} (${quantity})`;
}
```

<!-- expect(ingredientToString({name: 'Pizza', quantity: 6})).toBe('Pizza (6)') -->

Or like this:

```js
function ingredientToString({ name, quantity }) {
  return `${name} (${quantity})`;
}
```

<!-- expect(ingredientToString({name: 'Pizza', quantity: 6})).toBe('Pizza (6)') -->

I prefer the last one for the reasons I explain in the [Naming is hard](#naming-is-hard) chapter, but I wouldn’t ask another developer to change their code just because they use another option: they are all fine.

A few more examples below.

Old-style functions or arrow functions, explicit return or implicit return:

```js
function getDropdownOptions(options) {
  return options.map(option => option.value);
}
```

<!-- expect(getDropdownOptions([{value: 1}, {value: 2}])).toEqual([1, 2]) -->

Or:

```js
const getDropdownOptions = options =>
  options.map(option => option.value);
```

<!-- expect(getDropdownOptions([{value: 1}, {value: 2}])).toEqual([1, 2]) -->

Or the same with default export:

```js
const Button = props => <button className="Button" {...props} />;
export default Button;
```

<!--
const {container: c1} = RTL.render(<Button>Tacos</Button>);
expect(c1.textContent).toEqual('Tacos')
-->

Or:

```js
export default function Button(props) {
  return <button className="Button" {...props} />;
}
```

<!--
const {container: c1} = RTL.render(<Button>Tacos</Button>);
expect(c1.textContent).toEqual('Tacos')
-->

I can probably write a whole book of such examples, and let’s not forget the eternal debate of tabs vesus spaces.

In all the examples above I prefer the last variation but I’d never ask someone to change their code during code review if they use another variation.

There’s zero code readability improvement. The code is just different, none of the variations are better than the other. And even the consistency argument isn’t good enough, unless we can automate code replacement completely transparently for the developer. Otherwise the cost of maintaining the convention would be too high.

## Conclusion

My rule of thumb here is: automate or forget. For example, [Prettier](https://prettier.io/) (see the [Autoformat your code](#autoformat-your-code) chapter) formats code with almost zero config, which saves enormous amount of time while you write code, read someone else’s code or discuss code style in your team.

Be vigilant when you adapt [a popular code styles](https://blog.sapegin.me/all/javascript-code-styles/): many are too opinionated and want us to write code in a very specific way, even when it doesn’t improve readability or maintainability of the code, or reduce the number of bugs.

Prefer explicit over implicit, write code to maximize readability but don’t be too strict with others when they don’t do it the same way you would. Next time you review someone else’s code and want to ask them to change a piece of code, ask yourself: does it really make code more readable and maintainable or just makes it look more familiar to me. If it’s the latter, please don’t write that comment.
