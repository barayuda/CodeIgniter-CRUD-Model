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

Now the model is ready to do basic record retrieval (even paginated).

## Basic Record Retrieval

These simple examples won't exactly show you the power behind this CRUD model, but we do need to get the basics out of the way.

	// IN THE CONTROLLER

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

	// IN THE CONTROLLER

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

	// IN THE VIEW

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
		'prev_tag_close'	=> '</li>'
	);

One drawback to the pagination feature is it won't generate page links properly if it's being loaded on a page without the method name included in the URI. For example, if our paginated results are being generated from a controller named Blog and a method named index, then the URI must have the full index.php/blog/index, and not just index.php/blog.

## Optional Query Building

Many times, you may have a query which should be built and run the same way 99% of the time, but a bit differently 1% of the time. This CRUD model allows for us to easily specify when we want the query to run differently.

Consider the following model:

	class Mdl_Blog_Posts extends CRUD_Model
	{
		public $table = 'blog_posts';
		public $primary_key = 'blog_posts.post_id';

		public function num_comments()
		{
			$this->db->select('(SELECT COUNT(*) FROM blog_comments WHERE blog_comments.post_id = blog_posts.post_id) AS num_comments');
		}
	}

And consider the following controller code:

	// This will retrieve all records and will not pay any attention to the num_comments() method in the model
	$posts = $this->mdl_blog_posts->get()->result();

	// The results from the above query will be a standard select statement on the blog_posts table

	// This will tell the model to also include the additional clauses / conditions / statements when building the query:
	$posts = $this->mdl_blog_posts->get('num_comments')->result();

	// The results from the above query will be a standard select statement on the blog_posts table, along with the embedded subquery.

Optional query building methods can be named anything, and can be passed to any of the 3 get methods. They can include joins, wheres, selects, etc, etc.

## Form Validation

Let's add a new method into our model:

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

		public function validation_rules()
		{
			return array(
				'post_title' => array(
					'field' => 'post_title',
					'label' => 'Title',
					'rules' => 'required'
				),
				'post_content' => array(
					'field' => 'post_content',
					'label' => 'Content',
					'rules' => 'required'
				),
				'category_id' => array(
					'field' => 'category_id',
					'label' => 'Category'
				),
				'post_date' => array(
					'field' => 'post_date',
					'label' => 'Date',
					'rules' => 'required'
				)
			);
		}
	}

As you can see, validation rules are written into the model as a method which returns an array containing validation rules which are recognized by CodeIgniter.

To run validation:

	// IN THE CONTROLLER
	if ($this->mdl_blog_posts->run_validation())
	{
		// Save the data or whatever needs to be done
	}
	else
	{
		// Yell at the user for not filling the form in properly
	}

By default, the run_validation() method will look for a method in the child model called "validation_rules". However, the rule method doesn't have to be named "validation_rules", and there can be more than one. To specify a particular rule method:

	if ($this->mdl_blog_posts->run_validation('some_other_validation_rule_method'))
	{
		// Save the data or whatever needs to be done
	}

As long as the method is returning an array of CodeIgniter validation rules, that's all that matters.

The validation array is closely tied to how the CRUD model will handle the form data. Notice in the rule array above, the category_id field doesn't have any rules associated with it. This is because, in this example, that particular field is not required and does not need any special validation. It must exist in the array, however, so the CRUD model knows to include it when saving the record, or when re-populating the form.

## Saving Data

First, let's look at how a form method might be used from a controller to interact with a model which extends the CRUD model:

	public function form($id = NULL)
	{
		// Check to see if the form validates
		if ($this->mdl_blog_posts->run_validation()
		{
			// It validates -- save the record and redirect to index
			$this->mdl_blog_posts->save($id);
			redirect('blog/index);
		}

		// Prepare the form if it hasn't been submitted
		if (!$_POST)
		{
			$this->mdl_blog_posts->prep_form($id);
		}

		// Load the view
		$this->load->view('blog_form');
	}

On a closer look, there are really only two things which haven't yet been covered in this documentation.

	$this->mdl_blog_posts->prep_form($id);

The prep_form method should be executed only when the form has not yet been submitted (if (!$_POST)). If the $id variable contains a value, it will instruct the CRUD model to retrieve the appropriate record from the database and have the data ready to re-populate the form.

	$this->mdl_blog_posts->save($id);

The save method (obviously) saves the form data. The fields which it saves is dependent upon the validation rules (remember earlier we included the category_id field in the validation rule array even though it didn't have any special rules). The save method will perform an insert or an update accordingly, based on whether or not an $id is passed to it, and if the $id actually has a value or not.

Now that we've seen the general logic a controller form method might have, let's take a look at how the view might look:

	<form method="post">

		<label>Title</label>
		<input type="text" name="title" value="<?php echo $this->mdl_blog_posts->form_value('title'); ?>">

		<label>Date</label>
		<input type="text" name="post_date" value="<?php echo $this->mdl_blog_posts->form_value('post_date'); ?>">

		<label>Category</label>
		<select name="category_id">
			<?php foreach ($categories as $category) { ?>
			<option value="<?php echo $category->category_id; ?>" <?php if ($category->category_id == $this->mdl_blog_posts->form_value('category_id')) { ?>selected="selected"<?php } ?>><?php echo $category->category; ?></option>
			<?php } ?>
		</select>

		<!-- You can get the picture from the above 3 fields -->

	</form>

The form_value('field_name') method will return the value currently assigned to the specified field. If prep_form($id) was used to re-populate the form for an edit, then form_value('field_name') will return the original value from the database. If the form has been submitted and fails validation, then it will return the last submitted value.

Sometimes we need to manipulate our data a bit before it makes it from the form to the database. We'll override a method in the CRUD model to do this:

	// IN THE Mdl_Blog_Posts MODEL
	public function db_array()
	{
		// First, retrieve the default array
		$db_array = parent::db_array();

		// Next, manipulate as necessary
		$db_array['post_date'] = strtotime($db_array['post_date']);

		// Now, return the modified array
		return $db_array;
	}

That's it. Nothing additional has to be done in the controller - it's all handled in the model. The CRUD model's db_array method looks at the validation rules which were run and builds an array to save to the database based off of those rules. The elements in the array correspond to the field elements of the validation rules.

Let's do the same thing, but in reverse -- let's force the model to change the UNIX timestamp back into a human readable value so when it's re-populated back into the form, it's in the proper state:

	// IN THE Mdl_Blog_Posts MODEL
	public function prep_form($id)
	{
		// First, run the parent method
		parent::prep_form($id);

		// Next, manipulate as necessary
		$this->set_form_value('post_date', date('m/d/Y', $this->form_value('post_date'));
	}

Now, when an existing record is edited and the post_date value is output back into the form, it will be formatted appropriately. Once again, it's all handled in the model - nothing to do in the controller or the view.

## Summary

This documentation should provide adequate usage examples for this CRUD model. The CRUD model can be used in any CodeIgniter project, and tries to adhere to native CodeIgniter query builder standards without straying far off in some stupid subset of logic. I hope you find this to be useful in your projects as much as I've found it useful in mine.