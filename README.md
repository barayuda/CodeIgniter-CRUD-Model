CodeIgniter-CRUD-Model
======================

Extend this base model in CodeIgniter to make your database interactions super spiffy.

## Install, Extend & Conquer

To install, simply copy the file as CRUD_Model.php to the application/core folder of your CodeIgniter installation.

Then, instead of:

	class Your_Model extends CI_Model

Extend the CRUD Model:

	class Your_Model extends CRUD_Model

Once your model extends the CRUD model, you'll need to define two critical properties:

	class Your_Model extends CRUD_Model
	{
		public $table = 'the_table_name';
		public $primary_key = 'the_table_name.primary_key_column';
	}

Now the model is ready to do basic record retrieval (even paginated), inserts and updates and deletes (hence, the basic CRUD methods).

## Retrieve Records

## Perform Validation

## Save Data