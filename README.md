# Firestore to BigQuery export
NPM package for copying and converting [Cloud Firestore](https://firebase.google.com/docs/firestore/) data to [BigQuery](https://cloud.google.com/bigquery/docs/).

<p align="center">
  <a href="LICENSE">
    <img src="https://img.shields.io/badge/license-MIT-brightgreen.svg?" alt="Software License" />
  </a>
  <a href="https://npmjs.org/package/firestore-to-bigquery-export">
    <img src="https://img.shields.io/npm/v/firestore-to-bigquery-export.svg?" alt="Packagist" />
  </a>
  <a href="https://npmjs.org/package/firestore-to-bigquery-export">
    <img src="https://img.shields.io/npm/dm/firestore-to-bigquery-export.svg?" alt="Packagist" />
  </a>
  <a href="https://github.com/Johannes-Berggren/firestore-to-bigquery-export/issues">
    <img src="https://img.shields.io/github/issues/Johannes-Berggren/firestore-to-bigquery-export.svg?" alt="Issues" />
  </a>
</p>

Firestore is awesome. BigQuery is awesome. But transferring data from Firestore to BigQuery sucks.
This package lets you plug and play your way out of config hell.

- Create a BigQuery dataset with tables corresponding to your Firestore collections.
- Table schemas are automatically generated based on your document property data types.
- Convert and copy your Firestore collections to BigQuery.

This package doesn't write anything to Firestore.

## Contents
  * [Installation](#installation)
  * [How to](#how-to)
    + [API](#api)
    + [Examples](#examples)
  * [Limitations](#limitations)
  * [Issues](#issues)
  * [Issues](#to-do)

## Installation
> npm i firestore-to-bigquery-export

```javascript
import bigExport from 'firestore-to-bigquery-export'

// or

const bigExport = require('firestore-to-bigquery-export')

// then

const GCPSA = require('./Your-Service-Account-File.json')
bigExport.setBigQueryConfig(GCPSA)
bigExport.setFirebaseConfig(GCPSA)
```

## How to

### API
```javascript
bigExport.setBigQueryConfig(
  serviceAccountFile // JSON
)
```

```javascript
bigExport.setFirebaseConfig(
  serviceAccountFile // JSON
)
```


```javascript
bigExport.createBigQueryTable(
  datasetID, // String
  collectionName, // String
  verbose // boolean
)
// returns Promise<Array>
```

```javascript
bigExport.copyToBigQuery(
  datasetID, // String
  collectionName, // String
  snapshot // firebase.firestore.QuerySnapshot
)
// returns Promise<number>
```

```javascript
bigExport.deleteBigQueryTable(
  datasetID, // String
  tableName // String
)
// returns Promise<Array>
```


### Examples
```javascript
/* Create table 'account' in BigQuery dataset 'firestore'. You have to create the dataset beforehand.
 * The given table name has to match the Firestore collection name.
 * Table schema will be autogenerated based on the datatypes found in the collections documents.
 */
bigExport.createBigQueryTable('firestore', 'accounts')
    .then(res => { console.log(res) })
    .catch(error => console.error(error))
```

Then, you can transport your data:
```javascript
/* Copying and converting all documents in the given Firestore collection snapshot.
 * Inserting each document as a row in tables with the same name as the collection, in the dataset named 'firestore'.
 * Cells (document properties) that doesn't match the table schema will be rejected.
 */
firebase.collection('payments').get()
    .then(snapshot => bigExport.copyToBigQuery('firestore', 'payments', snapshot))
    .then(res => { console.log('Copied ' + res + ' documents to BigQuery.') })
    .catch(error => console.error(error))

/*
 * You can do multiple collections async, like this.
 * If you get error messages, you should probably copy fewer collections at a time.
 */
const collectionNames = ['payments', 'profiles', 'ratings', 'users']

Promise.all(collectionNames.map(n => {
    return firestore.collection(n).get()
      .then(c => bigExport.copyToBigQuery('firestore', n, c))
  }))
    .then(res => {
      console.log('Copied ' + res.reduce((a, b) => a + b) + ' documents to BigQuery.')
    })
    .catch(error => console.error(error))
```

After that, you may want to refresh your data. For the time being, the quick and dirty way is to delete your tables and make new ones:
```javascript
// Deleting the given BigQuery table.
bigExport.deleteBigQueryTable('firestore', 'accounts')
    .then(res => { console.log(res) })
    .catch(error => console.error(error))
```

## Keep in mind
* If there's even one prop value that's a FLOAT in your collection during schema generation, the column will be set to FLOAT.
* If there are ONLY INTs, the column will be set to INTEGER.
* All columns will be NULLABLE.

## Limitations
* Your Firestore data model should be consistent. If a property of documents in the same collection have different data types, you'll get errors.
* Patching existing BigQuery sets isn't supported (yet). To refresh your datasets, you can `deleteBigQueryTables()`, then `createBigQueryTables()` and then `copyCollectionsToBigQuery()`.
* Changed your Firestore data model? Delete the corresponding BigQuery table and run `createBigQueryTables()` to create a table with a new schema.
* When running this package via a Cloud Function, you may experience that your function times out if your Firestore is large, (Deadline Exceeded). You can then:
    * Increase the timeout for your Cloud Function in the [Google Cloud Platform Cloud Function Console](https://console.cloud.google.com/functions).
    * Run your function locally, using `firebase serve --only functions`. 

## Issues
Please use the [issue tracker](https://github.com/Johannes-Berggren/firestore-to-bigquery-export/issues).

## To-do
* Improve the handling of arrays.
* Implement patching of tables.
