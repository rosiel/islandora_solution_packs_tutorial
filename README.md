# islandora_solution_packs_tutorial
## What is a solution pack?
Solution packs are drupal modules that define, for a type of object, what happens throughout that  object's lifecycle. A solution pack defines what happens on:
* ingest - what forms are associated with this content type? What datastreams do they write to? Are there other steps during the ingest process? What other objects will this be associated with? 
* post-ingest - what derivatives are created?
* display - how does the object's page look?
* delete/purge - does the deletion cascade?

Objects may be simple or complex, but will always represent a digital asset in a meaningful way. 


## Required Objects

A solution pack implements ```hook_islandora_required_objects()``` to create two Fedora objects:
### Content Model Object 
To let Islandora recognize this new content model, the solution pack must ingest an object of type 'fedora-system:ContentModel-3.0'. The DS-COMPOSITE-MODEL datastream of this content model object defines what datastreams are required or expected for objects of this model. This definition is for illustration only; is not enforced (in 7.x).
### Default collection 
Islandora won't let you ingest an object without a collection configured to allow this content model. By convention, a solution pack will provide a collection with this content model in its Collection Policy.

## Form Associations
A solution pack should define the form to use for ingest/editing. This is given as an xml file and is registered using ```hook_islandora_xml_form_builder_form_associations()```. The association links this form to a specific datastream of this content model. 
