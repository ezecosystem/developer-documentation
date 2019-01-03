# Administration management

Once you set up your environment you can start your work as an administrator.
Your most useful tools can be found in **Admin Panel** and in drop-down menu under your user name.

## Systemn Information

The system information panel in Platform UI is implemented in a [ezsystems/ez-support-tools repository](https://github.com/ezsystems/ez-support-tools).
It allows you to add custom panels to the UI, or customize existing ones.
There you will also find basic system information such as versions of all installed packages. 

## Sections

Sections are used to divide Content items in the tree into groups that are more easily manageable by content editors.
Division into Sections allows you, among others, to set permissions for only a part of the tree.

Technically, a Section is a number, a name and an identifier.
Content items are placed in Sections by being assigned the Section ID, with one item able to be in only one Section.

When a new Content item is created, its Section ID is set to the default Section (which is usually Standard).
When the item is published it is assigned to the same Section as its parent. Because content must always be in a Section, unassigning happens by choosing a different Section to move it into.
If a Content item has multiple Location assignments then it is always the Section ID of the item referenced by the parent of the main Location that will be used.
In addition, if the main Location of a Content item with multiple Location assignments is changed then the Section ID of that item will be updated.

When content is moved to a different Location, the item itself and all of its subtree will be assigned to the Section of the new Location.
Note that it works only for copy and move; assigning a new section to a parent's content does not affect the subtree, meaning that Subtree cannot currently be updated this way.

Sections can only be removed if no Content items are assigned to them. Even then, it should be done carefully.
When a Section is deleted, it is only its definition itself that will be removed.
Other references to the Section will remain and thus the system will most likely lose consistency.

!!! warning

    Removing Sections may corrupt permission settings, template output and other things in the system.

Section ID numbers are not recycled. If a Section is removed, its ID number will not be reused when a new Section is created.

![Sections screen](img/admin_panel_sections.png)

## Roles and users

To give users an access to your website you need to assign them roles in the Admin panel.
Each role consists of:

**Policies**
 
Rules that give users access to different function in a module.
You can restrict what user can do with Limitations.
The available Limitations depend on the chosen Policy.
When Policy has more than one Limitation, all of them have to apply.
See [example below](#restrict-editing-to-part-of-the-tree).

!!! note

    Limitation specifies what a User can do, not what they can't do.
    A `Location` Limitation, for example, gives the User access to content with a specific Location, not prohibits it. See [Available Limitations](limitations.md#available-limitations) for further information.
 
**Assignments** 

After you created all Policies, you can assign the Role to users and/or groups with possible additional Limitations.
Every User or User Group can have multiple roles.
A User can also belong to many groups, for example, Administrators, Editors, Subscribers.

Best practice is to avoid assigning Roles to Users directly.
Model your content (types, sections, Locations etc.) in a way that can be accessed by generic roles.
That way system will be more secure and easier to manage. 
This approach also improves performance. Role assignments and Policies are taken into account during search/load queries.

See [Permissions](permissions.md) for further information.

### Use Cases

Here are a few examples of sets of Policies you can use to get some common permission configurations.

#### Enter back end interface

To allow the User to enter the Back Office interface and view all content, you need to set the following Policies:

- `user/login`
- `content/read`
- `content/versionread`
- `section/view`
- `content/reverserelatedlist`

These Policies will be necessary for all other cases below that require access to the content structure.

#### Create and publish content

To create and publish content, the user must additionally have the following Policies:

- `content/create`
- `content/edit`
- `content/publish`

This also lets the user copy and move content, as well as add new Locations to a Content item (but not remove them!).

#### Create content without publishing

This option can be used together with eZ Enterprise's content review options.
Using the following Policies, the User is able to create content, but can't publish it; instead, they must send it for review to another User with proper permissions (for example, senior editor, proofreader, etc.).

- `content/create`
- `content/edit`

Note that without eZ Enterprise this setup should not be used, as it will not allow the User to continue working with their content.

#### Restrict editing to part of the tree

If you want to let the User create or edit content, but only in one part of the content tree, you need to use Limitations.
Three Limitations that could be used here are `Section` Limitation, `Location` Limitation and `Subtree of Location` Limitation.

Let's assume you have two Folders under your Home: Blog and Articles.
You can let a User create content for the blogs, but not in Articles by adding a `Subtree of Location` Limitation on the Blog Content item.
This will allow the User to publish content anywhere under this Location in the structure.

A `Section` Limitation can be used similarly, but a Section does not have to belong to the same Subtree of Location in the content structure, any Locations can be assigned to it.

If you add a `Location` Limitation and point to the same Location, the User will be able to publish content directly under the selected Location, but not anywhere deeper in its Subtree of Location.

Note that when a Policy has more than one Limitation, all of them have to apply, or the Policy will not work.
For example, a `Location` Limitation on Location `1/2` and `Subtree of Location` Limitation on `1/2/55` cannot work together, because no Location can satisfy both those requirements at the same time.
If you want to combine more than one Limitation with the *or* relation, not *and*, you can split your Policy in two, each with one of these Limitations.

#### Multi-file upload

Creating content through multi-file upload is treated in the same way as regular creation.
To enable upload, you need you set the following permissions:

- `content/create`
- `content/read`
- `content/publish`

You can control what Content items can be uploaded and where using Limitations on the `content/create` and `content/publish` Policies.

A Location Limitation limits uploading to a specific Location in the tree. A Content Type Limitation controls the Content Types that are allowed.
For example, you can set the Location Limitation on a **Pictures** Folder, and add a Content Type Limitation
which only allows Content items of type **Image**. This ensures that only files of type `image` can be uploaded,
and only to the **Pictures** Folder.

#### Manage Locations

To add a new Location to a Content item, the Policies required for publishing content are enough.
To allow the User to remove a Location, you need to grant them the following Policies:

- `content/remove`
- `content/manage_locations`

Hiding and revealing Location requires one more Policy: `content/hide`.

#### Removing content

To send content to trash, the User needs to have the `content/remove` Policy.

To remove an archived version of content, the User must have the `content/versionremove` Policy.

Further manipulation of trash requires the `content/restore` Policy to restore items from trash, and `content/cleantrash` to completely delete all content from the trash.

#### Registering Users

To allow anonymous users to register through the `/register` route, you need to grant the `user/register` Policy to the Anonymous User Group.

#### Admin

To access the Admin in the Back Office the User must have the `setup/administrate` Policy.
This will allow the User to view the Languages and Content Types.

Additional Policies are needed for each section of the Admin.

##### System Information

- `setup/system_info` to view the System Information tab

##### Sections

- `section/view` to see and access the Section list
- `section/edit` to add and edit Sections
- `section/assign` to assign Sections to content

##### Languages

- `content/translations` to add and edit languages

##### Content Types

- `Content Type/create`, `Content Type/update`, `Content Type/delete` to add, modify and remove Content Types

##### Object States

- `state/administrate` to view a list of Object States, add and edit them
- `state/assign` to assign Objects States to Content

##### Roles

- `role/read` to view the list of Roles in Admin
- `role/create`, `role/update`, `role/assign` and `role/delete` to manage Roles

##### Users

- `content/view` to view the list of Users

Users are treated like other content, so to create and modify them the User needs to have the same permissions as for managing other Content items.

## Languages

eZ Platform offers the ability to create multiple translations of your website.
Which version is shown to a visitor depends on the way your installation is set up.
A new language version for the website can be added in the Admin Panel in the Languages tab.
Every new language must have a name and a language code, written in the xxx-XX format, for example eng-US etc.
The multilanguage system operates based on a global translation list that contains all languages available in the installation.
After adding a language you may have to reload the application to be able to use it.
Depending on your set up, additional configuration may be necessary for the new language to work properly, especially with SiteAccesses.
See [Languages](internationalization.md) for futher information.

## Content Types

[Content Types characteristic](content_model.md/#content-types)
[Creating Content Type](getting_started/first_steps.md/#create-a-content-type)

## Object States

Object states are user-defined states that can be assigned to Content items.
Object states are contained in groups.
If a state group contains any states, each Content item is automatically assigned a state from this group.

You can assign states to content in the Back Office in the Content item's Details tab.

![Assigning an Object state to a Content item](img/assigning_an_object_state.png)

By default, eZ Platform contains one Object state group: **Lock**, with states **Locked** and **Not locked**.

![**Lock** Object state](img/object_state_lock.png)

Object states can be used in conjunction with permissions, in particular with the [ObjectStateLimitation](permissions.md#objectstatelimitation).
Their specific use cases depend on your needs and the setup of your permission system.