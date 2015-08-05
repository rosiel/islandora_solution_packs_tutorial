# islandora_solution_packs_tutorial
## What is a solution pack?
Solution packs are drupal modules that define, for a type of object, what happens throughout that  object's lifecycle. A solution pack defines what happens on:
* ingest - what forms are associated with this content type? What datastreams do they write to? Are there other steps during the ingest process? What other objects will this be associated with? 
* post-ingest - what derivatives are created?
* display - how does the object's page look?
* delete/purge - does the deletion cascade?

Objects may be simple or complex, but will always represent a digital asset in a meaningful way. 

## Note on where to implement hooks
Implement `hook_foo()` with a function in your `solution_pack_name.module` file named `solution_pack_name_foo()`. If it's not in the `.module` file, it won't be seen by Drupal. If your module isn't enabled, it won't be seen either. 

## Required Objects

A solution pack implements ```hook_islandora_required_objects()``` to create two Fedora objects:

### Content Model Object 
To let Islandora recognize this new content model, the solution pack must ingest an object of type 'fedora-system:ContentModel-3.0'. The DS-COMPOSITE-MODEL datastream of this content model object defines what datastreams are required or expected for objects of this model. This definition is for illustration only; is not enforced (in 7.x).
### Default collection 
Islandora won't let you ingest an object without a collection configured to allow this content model. By convention, a solution pack will provide a collection with this content model in its Collection Policy.

## Form Associations
A solution pack should define the form to use for ingest/editing. This is a two-step process, similar to the GUI, where one needs to upload (and name) a form from an XML file, then create associations to content types and datastream. 
*  `hook_islandora_xml_form_builder_forms()` creates the form from an XML file and names it.  
*  `hook_islandora_xml_form_builder_form_associations()` creates an association linking this form to a specific datastream of this content model. 
Unlike forms that were uploaded through the GUI, forms that were registered in code cannot be deleted, and associations cannot be removed (though they can be disabled). You can see them in the XML form associations (admin/islandora/xmlform). 

## Ingest Steps
By default, the ingest steps for any Islandora object include
* an option to upload the object from a FOXML file
* selecting a content model (if the collection policy allows multiple)
* selecting a form (if the form associations associate multiple with the chosen cmodel)

But the solution pack needs provide the form to upload the OBJ (if applicable), or any other datastreams. It does this with `hook_islandora_ingest_steps()` or `hook_CMODEL_PID_islandora_ingest_steps()`. This points to a Drupal form which is, by convention, located in the solution pack's /include/CMODEL_upload.form.inc. That file includes the form definition (by Drupal's magic naming conventions, this is in a function with the name that is the same as the form_id), and what to do when the form is submitted (by Drupal's magic naming conventions, this is in a function named `FORM_ID_submit()`).

## Derivatives
Define what derivatives should be generated using `hook_islandora_CMODEL_PID_derivative()`. The actual code to create the derivatives should be in /includes/derivatives.inc.

## Display
Implement `hook_CMODEL_PID_islandora_view_object()`. If you don't, a default function will kick in, but this lets you configure how your objects' pages appear. This function returns a keyed array of _themed output_ (i.e. call `theme()` to generate html). This means you might want to use `hook_theme()` and define your own themes.

## Delete


