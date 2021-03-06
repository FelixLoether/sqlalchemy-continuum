Configuration
=============

Global and class level configuration
------------------------------------

All Continuum configuration parameters can be set on global level (manager level) and on class level. Setting an option at manager level affects all classes within the scope of the manager's class instrumentation listener (by default all SQLAlchemy declarative models).

In the following example we set 'store_data_at_delete' configuration option to False at the manager level.

::


    make_versioned(options={'store_data_at_delete': False})



As the name suggests class level configuration only applies to given class. Class level configuration can be passed to __versioned__ class attribute.


::


    class User(Base):
        __versioned__ = {
            'store_data_at_delete': False
        }


Versioning strategies
---------------------


Similar to Hibernate Envers SQLAlchemy-Continuum offers two distinct versioning strategies 'validity' and 'subquery'. The default strategy is 'validity'.


Validity
^^^^^^^^

The 'validity' strategy saves two columns in each history table, namely 'transaction_id' and 'end_transaction_id'. The names of these columns can be configured with configuration options `transaction_column_name` and `end_transaction_column_name`.

As with 'subquery' strategy for each inserted, updated and deleted entity Continuum creates new version in the history table. However it also updates the end_transaction_id of the previous version to point at the current version. This creates a little be of overhead during data manipulation.

With 'validity' strategy version traversal is very fast. When accessing previous version Continuum tries to find the version record where the primary keys match and end_transaction_id is the same as the transaction_id of the given version record. When accessing the next version Continuum tries to find the version record where the primary keys match and transaction_id is the same as the end_transaction_id of the given version record.


Pros:
    * Version traversal is much faster since no correlated subqueries are needed


Cons:
    * Updates, inserts and deletes are little bit slower


Subquery
^^^^^^^^

The 'subquery' strategy uses one column in each history table, namely 'transaction_id'. The name of this column can be configured with configuration option `transaction_column_name`.

After each inserted, updated and deleted entity Continuum creates new version in the history table and sets the 'transaction_id' column to point at the current transaction.

With 'subquery' strategy the version traversal is slow. When accessing previous and next versions of given version object needs correlated subqueries.


Pros:
    * Updates, inserts and deletes little bit faster than in 'validity' strategy

Cons:
    * Version traversel much slower



Column exclusion and inclusion
------------------------------

With `include` and `exclude` configuration options you can define which entity attributes you want to get versioned. By default Continuum versions all entity attributes except DateTime columns with default values. If you want to include this columns you have to pass them to `include`.


::


    class User(Base):
        __versioned__ = {
            'include': ['created_at']
        }

        id = sa.Column(sa.Integer, primary_key=True)
        name = sa.Column(sa.Unicode(255))
        created_at = sa.Column(sa.DateTime)


Sometimes you may have columns you want to exclude from the history classes. You may pass the column names to `exclude` option as follows:

::


    class User(Base):
        __versioned__ = {
            'exclude': ['picture']
        }

        id = sa.Column(sa.Integer, primary_key=True)
        name = sa.Column(sa.Unicode(255))
        picture = sa.Column(sa.LargeBinary)




Basic configuration options
---------------------------

Here is a full list of configuration options:

* base_classes (default: None)
    A tuple defining history class base classes.

* table_name (default: '%s_history')
    The name of the history table.

* transaction_column_name (default: 'transaction_id')
    The name of the transaction column (used by history tables).

* operation_type_column_name (default: 'operation_type')
    The name of the operation type column (used by history tables).

* relation_naming_function (default: lambda a: pluralize(underscore(a)))
    The relation naming function that is being used for generating the relationship names between various generated models.

    For example lets say you have versioned class called 'User'. By default Continuum builds relationship from TransactionLog with name 'users' that points to User class.

* track_property_modifications (default: False)
    Whether or not to track modifications at property level.

* modified_flag_suffix (default: '_mod')
    The suffix for modication tracking columns. For example if you have a model called User that has two versioned attributes name and email with configuration option 'track_property_modifications' set to True, Continuum would create two property modification tracking columns (name_mod and email_mod) for UserHistory model.

* store_data_at_delete (default: True)
    Whether or not to store data in history records when parent object gets deleted.


Example
::


    class Article(Base):
        __versioned__ = {
            'transaction_column_name': 'tx_id'
        }
        __tablename__ = 'user'

        id = sa.Column(sa.Integer, primary_key=True, autoincrement=True)
        name = sa.Column(sa.Unicode(255))
        content = sa.Column(sa.UnicodeText)


Customizing versioned mappers
-----------------------------

By default SQLAlchemy-Continuum versions all mappers. You can override this behaviour by passing the desired mapper class/object to make_versioned function.


::

    make_versioned(mapper=my_mapper)


Customizing versioned sessions
------------------------------


By default SQLAlchemy-Continuum versions all sessions. You can override this behaviour by passing the desired session class/object to make_versioned function.


::

    make_versioned(session=my_session)
