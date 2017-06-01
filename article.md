# Build robust applications: validate all your inputs

## What is input validation ?
Any web application can be schematized as follows :

![Schema](/schema.png)

Whatever your language or framework is, a web application is basically a set of routes your can consider as black boxes that take some inputs and deliver some outputs. Your code can be considered as the specifications of how the input will be transformed into the output.

Input validation is the process of ensuring that your program operates on clean, correct and useful data. Simply put, it is ensuring that your program knows how to react when given unexpected inputs, like missing parameters, strings when a number is expected or a png file when your application only supports jpg.

A robust application is a program that can handle every unexpected input cases gracefully.

## Why you should do it ?
“Don’t trust user input” is a key principle of web development. If your user can send faulty data to your application, you must assume that he will eventually do so. Your system should be prepared to that, so you absolutely have to validate every parameter (type and content) the user can send to your routes.

By doing so, you will avoid the following problems:

- Security breach like SQL injections.
- Send to the user unexpected stack traces that will damage your application image and potentially reveal part of your development stack (which is also a security breach).
- Accidentally create errors that will kill your application process and put it down entirely.
- Persist in the database dirty/missing/inconsistent data.

## Where you should do it ?

As stated above, each parameter send to each controller should be inspected. The only way to ensure this is doing it in the controller itself, in the Back-end of your application. To be clear, Front-end input validation is there to help your user to give correct input, but it cannot force him to do so. Only the Back-end can. So the structure of your controllers should roughly look like this:

```javascript
myController(input) {
  errors = validateInput(input);
  if(errors)
    return errors;

  output = myService.doSomeBusinessLogic(input);
  return output;
}
```

As you can see, validating the user input should be the very first step of your controller, before calling any business logic through your services.

## How should it be done ?

Of course, there no universal way to implement input validation. But I can give you an exemple of the cleanest way I’ve encounter to do so. I will use javascript-like syntax here, but it can be transposed to any language (As a matter of fact, this implementation was inspired by Atlassian JIRA, a Java-based application).

Imagine an article management application, with a REST controller used to change an article status (Draft, Published or Private) :

```javascript
const articleManager = require('ArticleManager');
const ARTICLE_STATUSES = ['Draft', 'Published', 'Private'];

ArticleController.updateStatus = function(articleId, status, response) {
  // Input validation step
  validationErrors = [];
  article = validateArticleId(articleId, validationErrors);
  status = validateStatus(status, validationErrors);

  if(validationErrors.length > 0) {
    response.statusCode = 400;
    return validationErrors;
  }

  // Business logic
  article.status = status;
  article.save();

  // Response
  return article;
};

function validateArticleId(articleId, errors) {
  if(!articleId) {
    errors.push(new Error('Missing parameter articleId'));
    return null;
  }

  if(!Number.isInteger(articleId)) {
    errors.push(new Error('Invalid parameter articleId. It should be an integer'));
    return null;
  }

  article = articleManager.findById(articleId);
  if(!article) {
    errors.push(new Error('No article found with id ' + articleId));
    return null;
  }

  return article;
}

function validateStatus(status, errors) {
  if(!status) {
    errors.push(new Error('Missing parameter status'));
    return null;
  }

  if(!ARTICLE_STATUSES.includes(status)) {
    errors.push(new Error('Invalid parameter status.'));
    return null;
  }

  return status;
}
```

As you can see, for each input, I have created a validation function that will inspect all the cases of potential error relative to this input and push it to an array. This way you can detect all the potential errors and send them to the user before beginning any business logic.
