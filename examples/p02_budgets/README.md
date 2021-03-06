
# Budgets example app

Demonstrates `data_ingest` with some basic [customizations](../../docs/custom.md)

## Non-default features

- [Metadata](../../docs/custom.md) is collected for each file upload.

- [Uniqueness is enforced](../../docs/custom.md) for two of the metadata fields

- [A Table Schema](../../docs/custom.md) enforces rules on the uploaded data

- Data is injected [into a database table](../../docs/custom.md) instead of to JSON files

## Configuring the project

Mostly, the project was created like the [default one](default.md), with these
exceptions:

- Created a `budget_data_ingest` project:

    python manage.py startapp budget_data_ingest

- Added [a form](budget_data_ingest/forms.py), overriding the default `UploadForm` with one that adds three fields of metadata to each file upload

- Subclassed the default `Upload` [Django model](budget_data_ingest/models.py) to specify that two of the metadata fields should be unique together (that is, the combination of the two fields must be unique)

- Added a `BudgetItem` [Django model](budget_data_ingest/models.py) to receive data inserted from the uploads

- Added [table_schema.json](table_schema.json), which will be used to validate uploads

- Additions/edits to `p02_budget/settings.py`:

    INSTALLED_APPS = [
        'django.contrib.admin',
        ...
        'budget_data_ingest',
        'data_ingest',
    ]

    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.postgresql',
            'NAME': 'budget_ingestor',
        }
    }

    DATA_INGEST = {
        'MODEL': 'budget_data_ingest.models.Upload',
        'FORM': 'budget_data_ingest.forms.UploadForm',
        'DESTINATION': 'budget_data_ingest.models.BudgetItem',
        'VALIDATION_SCHEMA': 'table_schema.json',
    }

## To run locally

Create a PostgreSQL database named `budget_ingestor`, run the inital migrations, and
create a user account.

    createdb budget_ingestor
    python manage.py migrate
    python manage.py shell -c "from django.contrib.auth.models import User; User.objects.create_user(
        'chris', 'chris@gsa.gov', 'publicservice')"

Run the server.

    python manage.py runserver

Visit http://localhost:8000/data_ingest/, login as `chris/publicservice`, and try uploading
some CSVs (like the provided [example](staff.csv)).