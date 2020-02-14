---
layout: docs
title: Examples of Using Client Functions
permalink: /documentation/test-api/obtaining-data-from-the-client/examples-of-using-client-functions.html
---
# Examples of Using Client Functions

This topic describes examples of using client functions in tests.

* [Check a Page URL](#check-a-page-url)
* [Obtain a Browser Alias Within a Test](#obtain-a-browser-alias-within-a-test)
* [Complex DOM Queries](#complex-dom-queries)

## Check a Page URL

When performing certain actions on a tested web page (clicking buttons or links), it can redirect a browser to another page and you need to check if the desired page is open. For example, you are testing your web application's login form and need to verify if a particular web page opens after a user logs in.

You can create a client function and use the [window.location.href](https://www.w3schools.com/jsref/prop_loc_href.asp) property within it to return the URL of a page that is open on the client and then verify if the actual page URL matches the expected one using an [assertion](../assertions/README.md). The sample below demonstrates how to do this.

```js
import { ClientFunction } from 'testcafe';

fixture `My Fixture`
    .page `http://devexpress.github.io/testcafe/example`;

//Returns the URL of the current web page
const getPageUrl = ClientFunction(() => window.location.href);

test('Check the page URL', async t => {
    await t
        .typeText('#developer-name', 'John Smith')
        .click('#submit-button') //Redirects to the 'Thank you' page
        .expect(getPageUrl()).contains('thank-you'); //Checks if the current page URL contains the 'thank-you' string
});
```

## Complex DOM Queries

Client functions can be helpful for complex DOM queries. For example, a tested page contains a table with some data, and you need to validate data from two columns only. To do this, obtain data from these columns, push them to an array and then compare the returned array with the expected one.

Below is a sample test for the [https://js.devexpress.com/](https://js.devexpress.com/) page that contains a grid. The sample demonstrates how to create a client function that returns an array containing data from the grid’s **Customer** and **Sale Amount** columns.

```js
import { ClientFunction } from 'testcafe';

fixture `Get sale amount`
    .page('https://js.devexpress.com/');

    const getSalesAmount = ClientFunction(() => {
        const grid      = document.querySelector('.dx-datagrid-rowsview');
        const rowCount  = grid.querySelectorAll('.dx-data-row').length;
        const sales     = grid.querySelectorAll('td:nth-child(3)');
        const customers = grid.querySelectorAll('td:nth-child(7)');

        const array = [];

        for (let i = 0; i < rowCount; i++) {
            array.push({
                sales: sales[i].textContent,
                customer: customers[i].textContent
            });
        }

        return array;
    });

test('My test', async t => {
    await t
        .expect(getSalesAmount()).eql([
            { sales: '$6,370', customer: 'Renewable Supplies' },
            { sales: '$4,530', customer: 'Apollo Inc' },
            { sales: '$1,110', customer: 'Johnson & Assoc' },
            { sales: '$6,600', customer: 'Global Services' },
            { sales: '$2,830', customer: 'Health Plus Inc' },
            { sales: '$6,770', customer: 'Gemini Stores' },
            { sales: '$1,460', customer: 'Discovery Systems' }
        ]);
});
```

You can use [TestCafe selectors](../selecting-page-elements/selectors/README.md) instead of the native `querySelector` and `querySelectorAll` methods. Pass these selectors to the client function's [dependencies](README.md#optionsdependencies) option and call them as regular functions. You can use chains of [selector methods](../selecting-page-elements/selectors/functional-style-selectors.md) instead of complex CSS strings for more transparent syntax.
