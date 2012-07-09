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

## Defining the Model

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
	// SELECT `blog_posts`.*, `categories`.*, `authors`.`author_name` 
	// FROM (`blog_posts`) 
	// JOIN `authors` AS authors ON `authors`.`author_id` = `blog_posts`.`author_id` 
	// JOIN `categories` AS categories ON `categories`.`category_id` = `blog_posts`.`category_id` 
	// ORDER BY `blog_posts`.`post_date` DESC

As you can see, our model is now becoming more of a definition of our data and its structure instead of a class with a bunch of stupid, jumbled methods which build queries and return results and all that junk. Much cleaner, simpler, and easier to read.

## Single Line Pagination

One thing I've always hated about CodeIgniter is the amount of redundant configuration required to make pagination work. Have a look at how simple the CRUD model makes pagination:

	$paged_results = $this->mdl_blog_posts->paginate()->result();

Wait, what? That's it? Yep. Here's a more complete example:

	// THE CONTROLLER
	// Load the model
	$this->load->model('mdl_blog_posts');

	// Retrieve the paged results
	$paged_results = $this->mdl_blog_posts->paginate()->result();

	// Prepare the data to send to the view
	$data = array(
		'results' => $paged_results,
		'page_links' => $this->mdl_blog_posts->page_links
	);

	// Load the view
	$this->load->view('blog', $data);

	// THE VIEW
	<?php foreach ($results as $result) { ?>
	<!-- Do whatever -->
	<?php } ?>
	
	<?php echo $page_links; ?>

The style of the pagination elements can be set by creating a config file with an array $config['pagination_style']. Here's an example:

	// Location: application/config/pagination_config.php
	$config['pagination_style'] = array(
		'first_link'		=> '&lsaquo;&lsaquo;',
		'next_link'			=> '&rsaquo;',
		'prev_link'			=> '&lsaquo;',
		'last_link'			=> '&rsaquo;&rsaquo;',
		'full_tag_open'		=> '<div class="pagination"><ul>',
		'full_tag_close'	=> '</ul></div>',
		'first_tag_open'	=> '<li>',
		'first_tag_close'	=> '</li>',
		'last_tag_open'		=> '<li>',
		'last_tag_close'	=> '</li>',
		'cur_tag_open'		=> '<li class="active"><a href="#">',
		'cur_tag_close'		=> '</a></li>',
		'next_tag_open'		=> '<li>',
		'next_tag_close'	=> '</li>',
		'prev_tag_open'		=> '<li>',
		'prev_tag_close'	=> '</li>',