---
layout: default
title: Create a Phone Numbers Table
parent: Migrations
nav_order: 4
---

# Create a Phone Numbers Table

In this section we'll make a `phone_numbers` table and populate it with some
data.  This new table will be related to the `people` table that we made
[previously](./create-a-people-table.html), so the migration will be somewhate
more involved.

We'll start by generating a new migration script.

```
formn m create create_table_phone_numbers
```

Then fill in the SQL, which creates a new `phone_numbers` table and relates it
to `people` using a `personID` column and constraint.

```javascript
'use strict';

module.exports = {
  /**
   * Run the migration.
   */
  up(dataContext) {
    const sql = `
      CREATE TABLE phone_numbers (
        phoneNumberID INT NOT NULL PRIMARY KEY AUTO_INCREMENT,
        phoneNumber VARCHAR(255) NOT NULL,
        type VARCHAR(255),
        personID INT NOT NULL,

        CONSTRAINT fk__phone_numbers__personID__people
          FOREIGN KEY (personID) REFERENCES people(personID)
          ON DELETE CASCADE)`;
    const params = {};

    console.log(sql);

    return dataContext
      .getExecuter()
      .query(sql, params); 
  },

  /**
   * Bring down a migration.
   */
  down(dataContext) {
    const sql    = `DROP TABLE phone_numbers`;
    const params = {};

    console.log(sql);

    return dataContext
      .getExecuter()
      .query(sql, params); 
  }
};
```

### Populate the Phone Numbers Table

As usual, start by making a new migration script.

```
formn m create insert_phone_numbers
```

Now let's add some dummy phone numbers for people.  We'll add three phone
numbers for "Joe Shmo" and one for "Rand al'Thor."  It may be tempting to hard
code the `personID` for each new record, but that's a bad idea.  Migrations
should never depend on generated identifiers because identifiers may be
different between environments.  For example, if you bring the `insert_people`
migration up and down a few times the identifiers will obviously continue to
increment.  That in mind, we'll find each person by name and then add
associated phone numbers.

```javascript
'use strict';

const people = [
  {
    firstName: 'Joe',
    lastName: 'Shmo',
    phoneNumbers: [
      {phoneNumber: '530-307-8810', type: 'mobile'},
      {phoneNumber: '916-200-1440', type: 'home'},
      {phoneNumber: '916-293-4667', type: 'office'}
    ]
  },
  {
    firstName: 'Rand',
    lastName: 'al\'Thor',
    phoneNumbers: [
      {phoneNumber: '666-451-4412', type: 'mobile'}
    ]
  },
];

module.exports = {
  /**
   * Run the migration.
   */
  async up(dataContext) {
    const results = [];

    for (let i = 0; i < people.length; ++i) {
      // Find the personID by firstName and lastName.
      const selPerson = `
        SELECT  p.personID
        FROM    people p
        WHERE   p.firstName = :firstName
          AND   p.lastName = :lastName`;
      const person = people[i];

      console.log(selPerson);
      console.log(person);

      const [rows] = await dataContext
        .getExecuter()
        .query(selPerson, person);
      const personID = rows[0].personID;

      // Insert each phone number.
      for (let j = 0; j < person.phoneNumbers.length; ++j) {
        const insPhone = `
          INSERT INTO phone_numbers (personID, phoneNumber, type)
          VALUES (:personID, :phoneNumber, :type)`;
        const phone = person.phoneNumbers[j];

        phone.personID = personID;

        console.log(insPhone);
        console.log(phone);

        const result = await dataContext
          .getExecuter()
          .query(insPhone, phone);

        results.push(result);
      }
    }

    return results;
  },

  /**
   * Bring down a migration.
   */
  async down(dataContext) {
    const results = [];

    // All phone numbers in a flattened array.
    const phones = people
      .reduce((phones, person) => phones
        .concat(person.phoneNumbers), []);

    for (let i = 0; i < phones.length; ++i) {
      const sql = `
        DELETE
        FROM   phone_numbers
        WHERE  phoneNumber = :phoneNumber
          AND  type = :type`;
      const params = phones[i];

      console.log(sql);
      console.log(params);

      const result = await dataContext
        .getExecuter()
        .query(sql, params);

      results.push(result);
    }

    return results;
  }
};
```

### Run the Migrations

Use the `up` sub-command to run the two new migrations.

```
formn m up
```

You should experiment with the `up` and `down` sub-commands a bit to ensure you
understand how they work.  All of the example migrations created in this section
are available in the [formn-example](https://github.com/benbotto/formn-example)
repository in the
[migrations](https://github.com/benbotto/formn-example/blob/master/migrations)
directory.

In the [next section](../models/) we'll use the Formn
CLI to generate entities for the `people` and `phone_numbers` tables.

