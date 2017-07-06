## The Story
- why Laravel Enso exists
- how did we approach the need
- what was born


## Under the hood 

### Authentication
 - the standard Laravel authentication is used, via email & password

### Authorization

 - application wide, checking user status: active/inactive. The inactive status prevents the user from making requests. The check is made for every request, via a middleware. If a user is made inactive while he's still logged in, at his next request he'll be logged out.
 - application-section wide, via the menu visibility, depending on the user's role. The users that don't have access to a certain menu, can't see it. The check is made for each menu redraw. This level doesn't block access to routes, it just affects the visibility of the menus.
 - application-section wide, depending on access to routes, which is tied to the user's role and the permissions for that role. If he doesn't have access a 403 response is given back and a `laravel.log` entry is made.The check is made for each request, via a middleware. 
 - content specific, via policies. The check is made locally, when and where policies are used.

### Middleware & Middleware Groups
  - for the routes within the application, the 'core' middleware group is applied
  - the 'core' group contains the middleware below, presented in the order they're applied: 
     - `verify-active-state` - checks users's status (active/inactive)
     - `action-logger` - logs for each request the user's id, route, url, the HTTP verb and the timestamps
     - `verify-route-access` - authorizes the access for a route
     - `impersonate` - starts and stops the [impersonation](https://github.com/laravel-enso/Impersonate) of a user, when needed
     - `set-language` - sets the user's chosen language (locale)

### Permission Groups
  - permissions are grouped together in groups, which have mainly a descriptive purpose
  - when naming them, the same name convention is used as in the case of permissions  e.g. `group1.group2` 
  - permission groups are not linked directly to any other entity - with the obvious exception of permissions - nor are they used directly  



### [Permissions](https://github.com/laravel-enso/PermissionManager) & Routes
  
  - inside `web.php` there is a route for each action within the application, and each route has the `name` attribute defined
  - for each route where we need to have authorization checks, we must define a permission, permission which need to have the same name as the route
  - routes and groups are nested, the resulting name looking something like `group1.group2.route`
  - for each request we check the existence of the link between the user's role and the permission for the request's route
  - permissions' attributes:
     - name: see above
     - description - is human readable and is used when displaying a user's action history (on his profile page)     
     - type - may be `read` or `write` and is an informative flag     
     - default - flag which lets us know if a permission needs to be automatically allotted to any new role we create
  - for routes where we don't need to have authorization checks, permissions are not mandatory, **BUT**
  - if we want to log and display the users' actions, permissions become necessary, as they're used when displaying statistics
    


### Owners, [Roles](https://github.com/laravel-enso/RoleManager) & Users
  - users represent the operators using the application
  - roles may be loosely considered groups of permissions
  - owners may be considered user groups, they are the user's owners
  - an owner may have many users
  - an owner may have many roles
  - an user can have **just one owner** and **just one role**
  - rolul unui user nu poate fi decat unul din rolurile alocate ownerului
  - the role of a user user may only be one of the roles available for the owner
  - when a user is created, one must choose an owner and then a role from the list of available roles
  - userii au status activ/inactiv, cei inactivi ne mai avand posibilitatea de a se loga in aplicatie
  - users have an active or inactive status, where inactive users cannot login into the application (but can set/reset their password) 

### [Menus](https://github.com/laravel-enso/MenuManager) & [Roles](https://github.com/laravel-enso/RoleManager)
  - a menu represents an element of the main menu of the application
  - the entire menu is dynamically generated for the UI
  - a menu element may have several roles attached and is rendered only for the users with the respective roles
  - menu element attributes:
      - `parent_id`, if he has a parent
      - `name`, which is translated and visible in the UI
      - `icon`, the icon classes, visible in the UI
      - `order`, is used for ordering the elements of the menu and is automatically generated when reordering the menu 
      - `link`, is the link that's going to be accessed when clicking on the menu
      - `has_children`, is a flag telling us if a menu element has children and is used when rendering the menu
      - when reordering the menu the `has_children` flag is automatically set as needed
  


### Breadcrumbs
  - are dynamically generated depending on the current route
  - the displayed text may be customized and, if custom values are missing, the default values are used
  - inserting breadcrumbs in a page may be easily done by including the `breadcrumbs` package that comes with the package  



### [Telemetry](https://github.com/laravel-enso/ActionLogger)
   - the `action-logger` middleware saves inside the `action_logs` table each user's action, recording the route, URL, HTTP verb and timestamp
   - the implicit `login` event that Laravel fires on a user's login triggers the save of the user's ip, user-agent and timestamp inside the `logins` table



### Exceptions    
   - where needed, `EnsoException` instances are thrown
   - depending on the type of the request (ajax or non ajax) a different response is given back   


### [Localisation](https://github.com/laravel-enso/Localisation)
The module allows having localized versions of the application.
- the `languages` table stores the available languages for localisation 
   - `name` - the language code, e.g. 'en'
   - `display_name` - the lable for the language, visible in the UI, e.g. 'English'
   - `flag` - the icon class used for showing the flag

- when translating, the new Laravel mechanism is used, respectively the function `__()` 
- the main language is considered to be english
- the keys are, by convention, in english and in a human readable format e.g. 'Date of Birth', and if a key is not found, the value of the key is used instead
- the keys and the values for the keys are kept in `resources/lang/*code*.json`  where code is the language code, e.g. 'de' for german, with the exception for the english language, since keys are already in english
- due to Laravel's implementation, there are 4 translation categories which cannot be implemented using the new mechanism: `auth`, `pagination`, `passwords`, `validation`. For this reason, we keep the respective language files in their proper language sub-folders
- the moment a new language is added from the interface
    - the new language is saved in the database    
    - the four php translation files are copied to a newly created language folder    
    - a new JSON language file is generated, containing the keys for the existing translations. The keys are collected using as reference the first existing JSON file
- when deleting a language
    - the language is removed from the database    
    - the language folder and its contents are removed    
    - the JSON language file is removed


### Preferences

The mechanism allows saving and loading upon login the user's preferences for a several aspects of the application.
- the preferences can be updated from the right-hand sidebar, where the user can also reset them as well as restart the tutorial function for the current page
- inside the `preferences` table, within `value` JSON type attribute, the following keys/settings are kept
    - `lang` - the user's language
    - `theme` - the currently selected theme
    - dtStateSave - flag for saving the state/preferences for each data-table within the application (for up to 90 days)
    - fixedHeader - the option of making the header fixed or scrollable
    - collapsedSidebar - flag for starting with / keeping the main left-hand navigation sidebar open or closed



### [Tutorials](https://github.com/laravel-enso/TutorialManager)
- the tutorial functionality may be restarted from the right-hand sidebar, using the `?` button 
- the `tutorials` table is used for the tutorial module and has several key attributes:
   - `permission_id` -  the permission where they're in use, since permissions are tied to routes, and we're using permissions to know which tutorials to load for a page
   - `element` - identifies the element within the DOM, and may be an id, in which case it should be prefixed with a `#` or a class, in which case it should be prefixed with `.`
   - `placement` -  sets the position of the tutorial dialog, relative to the DOM element, and can be: top, bottom, left sau right.
   - `order` - gives the order in which a particular tutorial element should be displayed, in the context of the available tutorials for a certain page.



### [Comments](https://github.com/laravel-enso/CommentsManager) & [Documents](https://github.com/laravel-enso/DocumentsManager) (optional modules)
   - polymorphic relationships are used, which makes it possible to attach comments and documents to any other entity
   - within the entity to which we want to attach comments/documents, we must use the respective trait (`Commentable` / `Documentable`)


   
### [Notifications](https://github.com/laravel-enso/Notifications) (optional module)
   - polymorphic relationships are used, in order to be able to attach notifications to any entity
   - for push notifications capabilities, the [Pusher](https://pusher.com/) service is used and integrated



### Environment
- .env & config
   - for push notifications, we use [Pusher](https://pusher.com/), therefore the following values must be set in the `.env` file:
      - BROADCAST_DRIVER=pusher
      - PUSHER_APP_ID=xxx
      - PUSHER_APP_KEY=xxx
      - PUSHER_APP_SECRET=xxx
   - within the configuration file `config/larave-enso.php` various options may be set, such as the folders used for storing uploads, avatars, etc. the caching duration and the timestamps format when displaying them



### [Datatable](https://github.com/laravel-enso/DataTable)

It uses several predefined elements in order to be able to quickly create and add a new table.

More details are available on the project's [repository page](https://github.com/laravel-enso/DataTable).



### [Select](https://github.com/laravel-enso/select)

It uses several predefined elements in order to be able to quickly add a new select.
It can handle having static option items or load them server-side.

More details are available on the project's [repository page](https://github.com/laravel-enso/select).



### [Cropping and optimizing images](https://github.com/laravel-enso/ImageTransformer)

It's handled automatically, when uploading pictures through the documents module or when adding an avatar for the user.
- external libraries are used
- it does graceful handling when the libraries are missing, logging the fact but allowing the upload.

More details are available on the project's [repository page](https://github.com/laravel-enso/ImageTransformer).



### [TrackWho](https://github.com/laravel-enso/TrackWho)

Collection of traits which permit the easy tracking of users who are creating, updating or soft deleting an entity

More details are available on the project's [repository page](https://github.com/laravel-enso/TrackWho).



### [Rememberable](https://github.com/laravel-enso/Rememberable)

Collection of traits which, together wth the [functionality](https://laravel.com/docs/5.4/cache) offered by Laravel, simplify using caching, 

More details are available on the project's [repository page](https://github.com/laravel-enso/Rememberable).



### [Log Manager](https://github.com/laravel-enso/LogManager)

Module meant for administrators which offers a streamlined interaction with the application logs.

More details are available on the project's [repository page](https://github.com/laravel-enso/LogManager).



### [History Tracker](https://github.com/laravel-enso/HistoryTracker)

Traits which allows creating snapshots of any model, with the purpose of keeping a history for that particular model.

More details are available on the project's [repository page](https://github.com/laravel-enso/HistoryTracker).


### [File Manager](https://github.com/laravel-enso/FileManager)

Module which is used internally to simplify working with files.

More details are available on the project's [repository page](https://github.com/laravel-enso/FileManager).



### [Avatar Manager](https://github.com/laravel-enso/AvatarManager)

Module which adds user avatar functionality.

More details are available on the project's [repository page](https://github.com/laravel-enso/AvatarManager).



### [Charts](https://github.com/laravel-enso/Charts)

Module which facilitates the process of adding charts in the application, for a greater impact when presenting data.

Is uses the [Chart.js](http://www.chartjs.org/) library.

More details are available on the project's [repository page](https://github.com/laravel-enso/Charts).



### [Contact Persons](https://github.com/laravel-enso/ContactPersons) (optional module)

Module which allows attaching contact persons to an owner.

More details are available on the project's [repository page](https://github.com/laravel-enso/ContactPersons).



### [Data Import](https://github.com/laravel-enso/DataImport) (optional module)

Module which permits importing data from the `xlsx` files. 

More details are available on the project's [repository page](https://github.com/laravel-enso/DataImport).



### [Structure Manager](https://github.com/laravel-enso/StructureManager)

Module which makes it easy to insert data in the application, useful when setting up modules. 

It can insert (and rollback) menus, permissions and permission groups.

More details are available on the project's [repository page](https://github.com/laravel-enso/StructureManager).


## How to

- creating a menu
- creating a permission
- creating permissions for a resource
- adding routes
- creating a page
- adding a server-side vue-select
- adding a data-table