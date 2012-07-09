CodeIgniter-CRUD-Model
======================

Extend this base model in CodeIgniter to make your database interactions super spiffy.

## Install, Extend & Conquer

For sake of simplicity, we'll pretend to work on a model which interacts with a table containing blog posts.

To install, simply copy the file as CRUD_Model.php to the application/core folder of your CodeIgniter installation.

Then, instead of:

	class Mdl_Blog_Posts extends CI_Model

Extend the CRUD Model:

	class Mdl_Blog_Posts extends CRUD_Model

Once your model extends the CRUD model, you'll need to define two critical properties:

	class Mdl_Blog_Posts extends CRUD_Model
	{
		public $table = 'blog_posts';
		public $primary_key = 'blog_posts.post_id';
	}

Now the model is ready to do basic record retrieval (even paginated), inserts and updates and deletes (hence, the basic CRUD methods).

## Basic Record Retrieval

These simple examples won't exactly show you the power behind this CRUD model, but we do need to get the basics out of the way.

	// IN YOUR CONTROLLER

	// Load the model as you normally would
	$this->load->model('mdl_blog_posts');

	// Retrieve all records into an object
	$all_records = $this->mdl_blog_posts->get()->result();

	// Retrieve all records into an array
	$all_records = $this->mdl_blog_posts->get()->result_array();

	// Retrieve a single record into an object
	$single_record = $this->mdl_blog_posts->get()->row();

	// Retrieve a single record into an array
	$single_record = $this->mdl_blog_posts->get()->row_array();

	// We can also chain CI's db methods through our model, just like using $this->db
	$records = $this->mdl_blog_posts->where('post_date >=', '2012-07-07')->order_by('post_date DESC')->get()->result();

Wait a minute... can't you do all of these examples by using CI's standard $this->db?

	$all_records = $this->db->get('blog_posts')->result()

Yes indeed, you could. However, up above, I told you that we needed to get the basics out of the way. Those were the basics, and now they're out of the way. On to the fun stuff.

## Advanced Record Retrieval

Let's take a look at the model code below:

	class Mdl_Blog_Posts extends CRUD_Model
	{
		public $table = 'blog_posts';
		public $primary_key = 'blog_posts.post_id';

		public function default_select()
		{
			$this->db->select('blog_posts.*, categories.*, authors.author_name'):
		}

		public function default_order_by()
		{
			$this->db->order_by('blog_posts.post_date DESC');
		}

		public function default_join()
		{
			$this->db->join('authors', 'authors.author_id = blog_posts.author_id');
			$this->db->join('categories', 'categories.category_id = blog_posts.category_id');
		}
	}

Now we'll jump back over to our controller:

	// Load the model
	$this->load->model('mdl_blog_posts');

	// Retrieve all the records
	$all_records = $this->mdl_blog_posts->get()->result();

	// This is the query the get() method from the line above would produce:
	// SELECT `blog_posts`.*, `categories`.*, `authors`.`author_name` FROM (`blog_posts`) JOIN `authors` AS authors ON `authors`.`author_id` = `blog_posts`.`author_id` JOIN `categories` AS categories ON `categories`.`category_id` = `blog_posts`.`category_id` ORDER BY `blog_posts`.`post_date` DESC

As you can see, our model is now becoming somewhat of a definition of our data and its structure instead of a class with a bunch of methods which build queries and return results and all that junk. Much cleaner, simpler, and easier to read.

