## How to

Before continuing, I'll assume that you're using the english language for the interface and that you're already logged in as an admin user.

### Creating a Menu



Let's start:
- Go to the System menu -> Menus, click on `Create Menu` and fill out
   - the desired name, for example 'My Contact Persons'
   - the icon class, for example 'fa fa-address-book  fa-fw'
   - the link that's going to get accessed, for example, 'myContactPersons'
- let's leave out parents, so we'll see the menu in the root, and set 'No' for children
- Save & and you should already be able to see the new menu
- Finally, set the roles, if you need to, otherwise admin should be selected as default.


If you click on the new menu, you get a 404 response, since the route doesn't exist. We'll do that next.

**Notes** 
- For an existent role, you may set the newly created menu as a default menu by accessing System -> Roles and editing the desired role.
- Once you've created a new menu or menus, don't forget to add the proper translations if you're using multiple languages
 
### Adding Routes and a Controller



Edit the routes file in your project folder `routes/web.php` and inside the main group, where you should already have 2 routes defined, 'home' & 'dashboard',
add a new route group:

```
    Route::group(['prefix' => 'myContactPersons', 'as' => 'myContactPersons.'], function () {
    
        Route::get('', 'MyContactPersonsController@index')->name('index');
    });
```

Now, if you click on the menu link, you'll get 500 error stating that the controller doesn't exist, so let's make one:
```
    php artisan make:controller MyContactPersonsController

```

and also create the index function inside:
```
    public function index()
    {
        return 'ok';
    }
```

If you try click on the menu link, now you'll not longer get an error but a toaster message letting you know you are not authorized for this action
We'll need to create a permission for the new route, and we'll do that next.

**Notes**

### Creating a Permission



Let's start.
First, let's create a new group for our permissions:
- Go to the System menu -> Permissions, click on `Create Group` and fill out
   - `name`, and it should coincide with the name we used for the new group in routes file, for example `myContactPersons`
   - `description`, here use anything you like, for example 'My Contact Persons'
   
Next, let's create the permissions:
- Go to the System menu -> Permissions, click on `Create Permission` and fill out
   - the route name for `name`, for example `myContactPersons.index`
   - for `description` use any description that makes sense for you, for example 'Index page for My Contact Persons'
   - for `type`, we'll use 'read', since we're not doing any write operations for this request
   - for `default access`, let's use 'Yes', since that means that any new role added will get by default access to this route
   - and for `Permission Group`, we should select the group we previously created, for example `myContactPersons`
- let's leave out parents, so we'll see the menu in the root, and set 'No' for children
- Save
- Finally, set the roles, if you need to, otherwise admin should be selected as default.

And now, we should be able to finally see the ok text we previously put in the index method.

### Creating Permissions for a Resource



We had previously created a single permission, and that's fine. 
Sometimes though, we might need to create permissions for a resource type of route, such as when declaring routes like:

```
    Route::resource('myContactPersons', 'MyContactPersonsController');
```

As a refresher, when doing this, Laravel under the hood declares a bunch of predefined routes, with names such as:
```
    myContactPersons.index
    myContactPersons.show
    myContactPersons.update
    etc.
```

You may list all the routes using `php artisan route:list`.

Of course, in our case, we would need to create permissions for each and every one, and we could that, one by one, or we can use the Enso built-in feature to do that in one step.

Before doing that, if you've followed along and also created the `myContactPersons.index` at the previous step, since the group resource creation would add a route for `index` as well,
we need to remove the previous permission:
- Go to the System menu -> Permissions, search for `myContactPersons` and click on the edit button
- remove all roles for the permission, and save the changes
- go back out and click on the delete button and confirm
   
Next, let's create the resource permissions:
- Go to the System menu -> Permissions, click on `Create Resource` and fill out
   - the `resource prefix`, for example in our case `myContactPersons`. If the resource group would be nested, we should define the whole path, say 'myOuterGroup.myContactPersons'   
   - for `Permission Group`, we should select the group we previously created, for example `myContactPersons`
- let's leave the `Data Tables` & `Vue Select` checkboxes unchecked for now.
- Save

If you want to see all the new permissions, filter the permissions by searching for `myContactPersons`.

Clicking on the menu should work as before, but in order to clean up the routes, let's edit `/routes/web.php` and 
- remove the old, manually added, index route. We'll keep the group empty for now.
- add the new resource route, as above.

It should look like this
```
    Route::group(['prefix' => 'myContactPersons', 'as' => 'myContactPersons.'], function () {
    
    });
    Route::resource('myContactPersons', 'MyContactPersonsController');
```

and work just as before.

**Note** The resource routes declarations should be *after* the declaration of the same group, if you also have the group defined separately.

In the next step, we're going to add a new page.

### Creating a Page

When creating a new page, you can look at existing page as an example or template. The dashboard `/views/dashboard/index.blade.php` is a good case.

If aiming for a consistent look, the page should have the following elements:
- it should extend the default layout: `@extends('laravel-enso/core::layouts.app')`
- it should include a title: @section('pageTitle', __("My Contact Persons"))
- it should include the content itself, within the content section `@section('content') @endsection`
- inside the 'content header' section, it should include the breadcrumbs template: `@include('laravel-enso/menumanager::breadcrumbs')`

    ```
        @section('content')
        
            <section class="content-header" v-cloak>
                @include('laravel-enso/menumanager::breadcrumbs')
            </section>
            <section class="content">
                <h1>My Contact Persons</h1>
            </section>
        
        @endsection
    ```

- it should also include some scripts, when needed within `@push('scripts') @endpush`

Let's put all that together and create a `views/myContactPersons/index.blade.php` view file. 
 
Of course, we need to update the `index` method inside the `MyContactPersonsController` and replace the return statement with `return view('myContactPersons.index');`

Take note that the breadcrumbs have been automatically generated for you, based on routes.

Next, we'll add some elements to our pages.

### Adding a Server-Side Vue-Select

If we want to add a select with a lot of option items, it is better to load them on request than dump all the data in the html page.
 
You can use the included VueJS powered vue-select component to achieve that.

In order to do that, we'll need to:
- add the `use SelectListBuilder` trait to our controller, which will make available the `getOptionsList()` method
- add the `protected $selectSourceClass=ContactPerson::class` to the controller, which lets it know what we're looking for when getting the select options
- since the trait is looking for the `name` attribute of any model declared as source, and our ContactPerson model doesn't have that property, we also need to declare `protected $selectAttribute='first_name'; ` as we'll use the first name for now
- next let's include the vue component in the page:

    ```
        <vue-select source="/myContactPersons/getOptionsList"
            name="contact_persons[]" multiple
            selected=""
            v-model="selected_contact_persons">
        </vue-select>
    ```
 - along with a data variable to bind the selection to:
   ```
   <script type="text/javascript"> 
   var vue = new Vue({ 
    el: '#app',
    data: {
     selected_contact_persons: []
    }
   }); 
   </script>
   ```

        If you go ahead and try it out, you'll notice that the request may get back a 500 response.
         The reason is that we haven't defined the route. 
 
 - let's add the required route inside the (previously empty) `myContactPersons` group: `Route::get('getOptionsList', 'MyContactPersonsController@getOptionsList')->name('getOptionsList');`
 
       If you refresh the page, you'll no longer get a 500 response, but a 403 one, 
       meaning we don't have the necessary permission.
       
 - let's add the permissions for the new route, from the menu, as we did before
 
 You may have noticed that we added a `multiple` flag for our vue-select component, and also for the name we defined as multiple value selection type (array).
 By removing the flag and then changing the name, we can turn it into a single select:
 
 ```
     <vue-select source="/myContactPersons/getOptionsList"
         name="contact_persons"
         selected=""
         v-model="selected_contact_persons">
     </vue-select>
 ```
 
 **Note** The resource routes declarations should be *after* the declaration of the same group, if you also have the group defined separately.
 
### Adding a Data-Table

I'll assume that you have already looked at the documentation for the DataTable module, as it can take a lot of options and parameters and I will not go over each and every one of them.

Let's start:
- first, we must include the `DataTable` trait inside our controller: `use DataTable;`. It provide the `initTable()` method, that gets called when the VueJS component will set itself up.
- next, we should declare the `protected $tableStructureClass = MyContactPersonsTableStructure::class;` that gets used by the trait, both when setting up the VueJS component and also when fetching and refreshing data.
- obviously, we need to   