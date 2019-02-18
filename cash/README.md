# cash

### Utility
The library cash is used to convert a value (1 by reference) from US Dollars ($) into Euro (EURO), British Pound Sterling (GBP) and Japanese yen (YEN).

# Install 

```
$ npm install i
```




## Usage
```js
#!/usr/bin/env node

'use strict';
const cash = require('./cash.js');

//we got all the libraries that are needed to run the code
const got = require('got');
const money = require('money');
const chalk = require('chalk');
const ora = require('ora');
const currencies = require('../lib/currencies.json');

const {API} = require('./constants'); //get the library where the url can be found

const cash = async command => { //we use an async command to make the function wait each time at each request the result
	const {amount} = command;
	const from = command.from.toUpperCase(); 
	const to = command.to.filter(item => item !== from).map(item => item.toUpperCase());

	console.log();
	const loading = ora({
		text: 'Converting...',
		color: 'green', //using the green color to get something
		spinner: {
			interval: 150,
			frames: to
		}
	});

	loading.start();

	await got(API, {//get the url element to load the information by using a require request
		json: true
	}).then(response => {
		money.base = response.body.base;//get from json the attribute in the body bracket to have the "base" element
		money.rates = response.body.rates; //get from json the attribute in the body bracket to have the "rate" element 

		to.forEach(item => { //for each element in the list item
			if (currencies[item]) {
				loading.succeed(`${chalk.green(money.convert(amount, {from, to: item}).toFixed(3))} ${`(${item})`} ${currencies[item]}`); //convert the amount givent into different currencies
			} else {
				loading.warn(`${chalk.yellow(`The "${item}" currency not found `)}`); //if the loading process was not a success then there is no currency conversion available
			}
		});

		console.log(chalk.underline.gray(`\nConversion of ${chalk.bold(from)} ${chalk.bold(amount)}`)); //Diplay the result in green with the initial amount to the currencies values
	}).catch(error => {//in case of an error occurs
		if (error.code === 'ENOTFOUND') { 
			loading.fail(chalk.red('Please check your internet connection!\n'));//message displayed to let the user know that the internet connection is bad
		} else {
			loading.fail(chalk.red(`Internal server error :(\n${error}`));//otherwise it's an error that is not linked with the internet connection
		}
		process.exit(1);
	});
};

module.exports = cash;//exporting the library to use it elsewhere

# Disclaimer
