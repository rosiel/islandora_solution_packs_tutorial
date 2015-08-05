# islandora_solution_packs_tutorial
## What is a solution pack?
Solution packs are Drupal modules that define, for a type of object, what happens throughout that  object's lifecycle. A solution pack defines what happens on:
* ingest - what forms are associated with this content type? What datastreams do they write to? Are there other steps during the ingest process? What other objects will this be associated with? 
* post-ingest - what derivatives are created?
* display - how does the object's page look?
* delete/purge - does the deletion cascade?

Objects may be simple or complex, but will always represent a digital asset in a meaningful way. Example: https://github.com/Islandora/islandora_solution_pack_image 

## Note on where to implement hooks
Implement `hook_foo()` with a function in your `solution_pack_name.module` file named `solution_pack_name_foo()`. If it's not in the `.module` file, it won't be seen by Drupal. If your module isn't enabled, it won't be seen either. 

## Required Objects

Before a solution pack can define behaviours relating to a content type, it needs to create two foundational objects in the Fedora repository - a content model object, and a collection to put them in. This is done using ```hook_islandora_required_objects()```.

### Content Model Object 
To let Islandora recognize this new content model, the solution pack must ingest an object of type 'fedora-system:ContentModel-3.0'. The DS-COMPOSITE-MODEL datastream of this content model object defines what datastreams are required or expected for objects of this model. This definition is for illustration only; is not enforced (in 7.x).
### Default collection 
Islandora won't let you ingest an object without a collection configured to allow this content model. By convention, a solution pack will provide a new collection (within islandora:root) with this content model in its Collection Policy. Users aren't necessarily expected to put their objects in this collection, but it's a courtesy so they can start using the content model immediately, and so that something shows up in the Islandora Repository to signify that this solution pack has been enabled.

To have these objects automatically created when the module is enabled, use a `.install` file and implement `hook_install()` and `hook_uninstall()` to call `islandora_install_solution_pack()`.  

## Form Associations
A solution pack should define the metadata form to use for ingest/editing. This is a two-step process, similar to doing this in the GUI, where one needs to upload (and name) a form from an XML file, then create associations to content types and datastream. 
*  `hook_islandora_xml_form_builder_forms()` creates the form from an XML file and names it.  
*  `hook_islandora_xml_form_builder_form_associations()` creates an association linking this form to a specific datastream of this content model. 
Unlike forms that were uploaded through the GUI, forms that were registered in code cannot be deleted, and associations cannot be removed (though they can be disabled). You can see them in the XML form associations (admin/islandora/xmlform). 

## Ingest Steps
By default, the ingest steps for any Islandora object include
* an option to upload the object from a MARCXML file
* selecting a content model (if the collection policy allows multiple)
* selecting a form (if the form associations associate multiple with the chosen cmodel)

But the solution pack needs provide the form to upload the OBJ (if applicable), or any other datastreams (like TN). It does this by implementing `hook_islandora_ingest_steps()` or `hook_CMODEL_PID_islandora_ingest_steps()`. This points to a Drupal form which is, by convention, located in the solution pack's /include/CMODEL_upload.form.inc. That file includes the form definition (by Drupal's magic naming conventions, this is in a function with the name that is the same as the form_id), and what to do when the form is submitted (by Drupal's magic naming conventions, this is in a function named `FORM_ID_submit()`).

## Derivatives
Define what derivatives should be generated using `hook_islandora_CMODEL_PID_derivative()`. The actual code to create the derivatives should be in /includes/derivatives.inc.

## Display
Implement `hook_CMODEL_PID_islandora_view_object()`. This function returns a keyed array of _themed output_ (i.e. call `theme()` to generate html). This means you might want to use `hook_theme()` and define your own themes. If you don't implement this hook, a default function `islandora_default_islandora_view_object()` will kick in and show some basic metadata, but this lets you configure how your objects' pages appear. 

## Configuration
If you want to define variables specific to your solution pack, define an admin menu path.

## Delete
If you have a complex object model, such as a Book or Newspaper, you may want to let the users delete an object _and all its child_ objects. Only do this if you're certain that child objects can't have multiple parents (or provide logic to mitigate that). An example is how islandora_solution_pack_newspaper implements `hook_form_islandora_object_properties_form_alter()`.

