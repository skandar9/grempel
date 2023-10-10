Project overview:

A control panel for a file management system accessible through a website allows users to create, edit, and download PDF files, as well as organize them into folders.



> :warning: **Warning:** This contents below ↓ contains just parts of my code.
>                        You can access my full project files by clone it from my GitLab repository
>                        (requires asking for my permissions to grant you access to it):
>                        https://gitlab.com/skandar.s1998/grempel 


## Contents
(contains descriptive parts of my code)

[Login to the dashboard functionality](#dashboard-login)

[Create file modal](#create-file-modal)

[loadPdf function](#loadPdf-function)

[Store file function](#store-file-function)

[Download file function](#download_file-function)
  
### **dashboard-login**

![App Logo](/images/login.png)

I used Laravel UI package in this project that provides a convenient way to generate the frontend boilerplate code for authentication views, such as login, registration, and password reset forms.

I Run the following command to install the package:

```shell
composer require laravel/ui
```

This package I used to generate the authentication views.Specifically, I utilized the following file to facilitate the login process for the admin, granting access to their dashboard.

`resources/views/auth/login.blade.php`: This file contains the HTML template for the login form. It includes form fields for the username and password, along with any necessary validation error messages.

I depended on `Auth` Facade that is a core component of Laravel that provides convenient methods for user authentication. It is used to authenticate users and manage their sessions.

### login functionality workflow:

## Routes in `web.php`

`routes\web.php`
### Index Route

```php
Route::get('/', [PagesController::class, 'index'])->name('index');
```
This route is responsible for handling the homepage of the application. When a user visits the root URL, the `index` method of the `PagesController` will be executed.

### Dashboard Route

```php
Route::get('/dashboard', [PagesController::class, 'dashboard'])->middleware(['auth'])->name('dashboard');
```
This route represents the dashboard page of the application. It is protected by the `'auth'` middleware, which means only authenticated users can access it. When a user visits the `/dashboard` URL, the `dashboard` method of the `PagesController` will be executed.


## PagesController

`app\Http\Controllers\PagesController.php`

```php
class PagesController extends Controller
{
    public function index(){
        return redirect('/login');
    }

    public function dashboard(){
        return view('dashboard');
    }
}
```
### `index` Method

This method is responsible for handling the request to the homepage of the application. It performs a redirect to the `/login` URL. When a user visits the homepage, they will be redirected to the login page.

## Authentication Route

`routes\auth.php`

```php
Route::get('/login', [AuthenticatedSessionController::class, 'create'])
                ->middleware('guest')
                ->name('login');

Route::post('/login', [AuthenticatedSessionController::class, 'store'])
                ->middleware('guest');
```
### Login Route
This route is responsible for rendering the login form. When a GET request is made to the `/login` URL, the `create` method of the `AuthenticatedSessionController` will be executed. This route is protected by the `'guest'` middleware, which ensures that only non-authenticated users can access the login page.

### Login POST Route
This route is responsible for handling the submission of the login form. When a POST request is made to the `/login` URL, the `store` method of the `AuthenticatedSessionController` will be executed. This route is also protected by the `'guest'` middleware.

## AuthenticatedSessionController

`app\Http\Controllers\Auth\AuthenticatedSessionController.php`

```php
class AuthenticatedSessionController extends Controller
{
    public function create()
    {
        return view('auth.login');
    }

    public function store(LoginRequest $request)
    {
        $request->authenticate();

        $request->session()->regenerate();

        return redirect()->intended(RouteServiceProvider::HOME);
    }
}
```
### `create` Method
This method is responsible for displaying the login view. It returns the view named `'auth.login'`, which is the login form view. When this method is called, the login form will be rendered and displayed to the user.

### `store` Method
This method handles the incoming authentication request when the login form is submitted. It expects an instance of `LoginRequest` as a parameter, which contains the validation rules for the login request. The method calls the `authenticate` method on the request instance to perform the authentication. If the authentication is successful, the user's session is regenerated, and the user is redirected to the intended page (defined by `RouteServiceProvider::HOME`).

## Login form view

`resources\views\auth\login.blade.php`

```html
<x-layouts.guest>
    <x-auth-card>
        <x-slot name="logo">
            <a href="/">
                <img src="/img/aim.png" width="220px">
            </a>
        </x-slot>

        <!-- Session Status -->
        <x-auth-session-status class="mb-4" :status="session('status')" />

        <!-- Validation Errors -->
        <x-auth-validation-errors class="mb-4" :errors="$errors" />

        <form method="POST" action="{{ route('login') }}">
            @csrf

            <!-- Username -->
            <div>
                <x-label for="username" :value="__('Username')" />
                <x-input id="username" class="block mt-1 w-full" type="text" name="username" :value="old('username')" required autofocus />
            </div>

            <!-- Password -->
            <div class="mt-4">
                <x-label for="password" :value="__('Password')" />
                <x-input id="password" class="block mt-1 w-full"
                                type="password"
                                name="password"
                                required autocomplete="current-password" />
            </div>

            <!-- Remember Me -->
            <div class="block mt-4">
                <label for="remember_me" class="inline-flex items-center">
                    <input id="remember_me" type="checkbox" class="rounded border-gray-300 text-indigo-600 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50" name="remember">
                    <span class="ml-2 text-sm text-gray-600">{{ __('Remember me') }}</span>
                </label>
            </div>

            <div class="flex items-center justify-end mt-4">
                @if (Route::has('password.request'))
                    <a class="underline text-sm text-gray-600 hover:text-gray-900" href="{{ route('password.request') }}">
                        {{ __('Forgot your password?') }}
                    </a>
                @endif

                <x-button class="ml-3">
                    {{ __('Log in') }}
                </x-button>
            </div>
        </form>
    </x-auth-card>
</x-layouts.guest>
```

The `login.blade.php` file contains the HTML markup for the login form view.
This file is installed with the package `laravel/ui`.

[🔝 Back to contents](#contents)

## LoginRequest

`app\Http\Requests\Auth\LoginRequest.php`

The `LoginRequest` class extends the `FormRequest` class in Laravel and is responsible for handling validation and authentication for the login form submission.
### `authorize` Method
This method determines if the user is authorized to make the login request. In this case, it always returns `true`, allowing any user to make the request.

### `rules` Method
This method defines the validation rules for the login request. It specifies that the `username` and `password` fields are required and must be of type `'string'`.

### `authenticate` Method
This method attempts to authenticate the user using the provided credentials. It first ensures that the request is not rate limited by calling the `ensureIsNotRateLimited` method. Then, it uses the `Auth::attempt` method to attempt authentication using the `username` and `password` fields from the request. It also includes the `remember` value as a boolean parameter to determine if the user wants to be remembered for future sessions. If the authentication attempt fails, the method hits the rate limiter by calling `RateLimiter::hit` and throws a `ValidationException` with the error message `'auth.failed'`. If the authentication attempt succeeds, the rate limiter is cleared by calling `RateLimiter::clear`.

The `LoginRequest` class handles the validation of the login form inputs and performs the authentication logic, including rate limiting to protect against brute-force attacks.

[🔝 Back to contents](#contents)

### **create-file-functionality**

[🔝 Back to contents](#contents)

### **create-file-modal**

![App Logo](/images/files.png)

`resources\views\dashboard\file_view.blade.php`

```html
<x-layouts.dashboard>
.
.
.
    <x-slot name="head">
        <SCript>
            var loadPdf = function(event) {
                var name = event.target.files[0].name;
                name = name.substr(0, name.lastIndexOf('.'));
                document.getElementById('pdf-title').value = name;
            }
        </SCript>
    </x-slot>
.
.
.
    <div class="d-flex">
        <button type="button" class="btn btn-success" style="margin-left: 900px;display:block;" data-toggle="modal"
            data-target="#addModal">{{ __('custom.Add') }}</button>
        <a href="{{ route('user_folders.folders.index', $folder->user_id) }}">
            <button type="submit" class="btn btn-dark" style="margin-left: 20px">{{ __('custom.Back') }}</button></a>
    </div>
    <!-- Add Modal -->
    <div class="modal fade" id="addModal" tabindex="-1" role="dialog" aria-labelledby="addModalTitle"
        aria-hidden="true">
        <div class="modal-dialog modal-dialog-centered" role="document">
            <div class="modal-content">
                <div class="modal-header">
                    <h5 class="modal-title" id="addModalLabel">{{ __('custom.Add File') }}</h5>
                    <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                        <span aria-hidden="true">&times;</span>
                    </button>
                </div>
                <div class="modal-body">
                    <div id="add-error" class="alert alert-danger" style="color: red;display:none;">
                        {{ __('custom.The entered values ​​are not valid') }}</div>
                    <form method="post" action="{{ route('folder_files.files.store', [$folder->id]) }}"
                        autocomplete="off" class="form-horizontal" enctype="multipart/form-data">
                        @csrf
                        <div class="row">
                            <label class="col-sm-3 col-form-label" for="input_file"
                                style="margin-top: 40px">{{ __('custom.File') }}</label>
                            <div class="col-sm-8">
                                <div class="form-group{{ $errors->has('file') ? ' has-danger' : '' }}">
                                    <input type="file" onchange="loadPdf(event)"
                                        class="form-control{{ $errors->has('file') ? ' is-invalid' : '' }}"
                                        name="file" id="input_file" />
                                    <input type="file" name="custom-file-input" size="40"
                                        class="wpcf7-form-control wpcf7-file" accept=".pdf" aria-invalid="false">
                                    <label for="input_file" class="btn btn-primary mt-5 ml-3">Select File</label>
                                    &nbsp;&nbsp;
                                    <img id="file" class="rounded"
                                        style="height:100px;width:100px;object-fit: cover; display:none">
                                </div>
                            </div>
                        </div>
                        <div class="row">
                            <label class="col-sm-3 col-form-label" for="input-text-en">{{ __('custom.Title') }}</label>
                            <div class="col-sm-8">
                                <div class="form-group{{ $errors->has('title') ? ' has-danger' : '' }}">
                                    <input class="form-control{{ $errors->has('title') ? ' is-invalid' : '' }}"
                                        name="title" id="pdf-title" type="text" required="true" accept=".pdf"
                                        aria-required="true" />
                                    @if ($errors->has('title'))
                                        <span id="title-error" class="error text-danger"
                                            for="input-title">{{ $errors->first('title') }}</span>
                                    @endif
                                </div>
                            </div>
                        </div>
                        <div class="modal-footer" style="border-top:none;">
                            <button type="submit" class="btn btn-success">{{ __('custom.Add') }}</button>
                        </div>
                    </form>
                </div>
            </div>
        </div>
    </div>
    <!-- End Modal -->
.
.
.
</x-layouts.dashboard>
```

1. Triggering the Modal:
   - The modal is triggered by clicking on a button with the attribute `data-toggle="modal"` and `data-target="#addModal"`.
   - In this case, the button with the class `btn-success` is responsible for triggering the modal.

2. Modal Structure:
   - The modal has a specific ID (`addModal`) which is used as the `data-target` attribute in the triggering button.
   - It is a bootstrap modal, indicated by the presence of the classes `modal`, `fade`, and `modal-dialog-centered`.
   - The modal consists of a header, body, and footer sections.

3. Header Section:
   - The header section contains a close button (`x` icon) to dismiss the modal.
   - It also includes a title for the modal, which is displayed as `{{ __('custom.Add File') }}`.

4. Body Section:
   - The body section includes a form for adding a file.
   - The form has a specific action (`{{ route('folder_files.files.store', [$folder->id]) }}`) where the file will be submitted to upon clicking the "Add" button.
   - The form uses the POST method.
   - Inside the form, there are two input fields:
     - The first input field is of type "file" with the name "file" and an ID of "input_file". It allows users to select a file from their device.
     - The second input field is also of type "file" with the name "custom-file-input" and a size of 40. It is hidden and used for styling purposes.
   - There is a label with the text "Select File" positioned next to the first input field. Clicking this label triggers the file selection dialog.
   - The image with the ID "file" is initially hidden and will be used to display a preview of the selected file.
   - There is also an input field for the title of the file with the ID "pdf-title".

5. File Selection and Title Handling:
   - The `onchange` event of the first file input field (`onchange="loadPdf(event)"`) triggers the [loadPdf](#loadPdf)
     function defined in the embedded script.
   - The `loadPdf` function extracts the name of the selected file, removes the file extension, and sets the value of the input field with the ID "pdf-title" to the extracted name.
   - This means that when a file is selected, its name will be displayed automatically in the title input field.

6. Error Handling:
   - There is an alert with the ID "add-error" that is initially hidden. It will be displayed if there are any validation errors when submitting the form.

7. Footer Section:
   - The footer section contains a single button with the class `btn-success`. Clicking this button triggers the form submission.
   - The text on this button is `{{ __('custom.Add') }}`.

[🔝 Back to contents](#contents)

### **loadPdf-function**

loadPdf function defines with this part from previous page:
```html
        <SCript>
            var loadPdf = function(event) {
                var name = event.target.files[0].name;
                name = name.substr(0, name.lastIndexOf('.'));
                document.getElementById('pdf-title').value = name;
            }
        </SCript>
```
The `loadPdf` function is a JavaScript function. It is responsible for handling the event when a file is selected using the file input field with the ID "input_file".
It extracts the name of the selected file, removes the file extension, and sets the value of the input field with the ID "pdf-title" to the modified file name

1. Function Definition:
   ````
   var loadPdf = function(event) {
      // Function body
   }
   ```

2. Event Parameter:
   - The function takes an `event` parameter, which represents the event object triggered by the file selection.
   - This event object contains information about the selected file(s) and can be accessed using its properties.

3. Extracting File Name:
   ````
   var name = event.target.files[0].name;
   ```
   - The `event.target.files` property returns an array-like object containing the selected files.
   - Since we are interested in the first selected file, we access it using `[0]`.
   - The `name` property of the file object provides the name of the selected file.

4. Removing File Extension:
   ````
   name = name.substr(0, name.lastIndexOf('.'));
   ```
   - The `lastIndexOf` method is used to find the position of the last occurrence of the dot (.) character in the file name.
   - The `substr` method is then used to extract the substring from the beginning of the file name up to the last occurrence
     of the dot    (.), effectively removing the file extension.
   - The modified file name is stored back in the `name` variable.

5. Setting Value of Input Field:
   ````
   document.getElementById('pdf-title').value = name;
   ```
   - The `getElementById` method is used to retrieve the input field element with the ID "pdf-title".
   - The `value` property of the input field is then set to the modified file name, which updates the displayed value in the input field.

[🔝 Back to contents](#contents)

### **store-file-function**

`app\Http\Controllers\Dashboard\FolderFileController.php`

```php
public function store(Request $request, $id)
{
    if (Auth::user()->role != 'admin')
        abort(403);

    $request->validate([
        'file'           => ['required', 'mimes:pdf'],
        'title'          => ['required', 'string'],
    ]);

    $folder = Folder::find($id);
    $file  = upload_file($request->file, 'file', 'qr_files');

    File::create([
        'folder_id'  => $folder->id,
        'file'       => $file,
        'title'      => $request->title,
    ]);

    return back()->withStatus('File is created successfully');
}
```
The `store` function ensures that the user has the 'admin' role, validates the request data, retrieves the associated folder, uploads the file, creates a new record in the 'File' model, and finally redirects the user back to the previous page with a success message.

1. Function Signature:
   ```
   public function store(Request $request, $id)
   ```
   - The `store` function accepts two parameters:
     - `$request`: An instance of the `Request` class, which contains the data submitted through the HTTP request.
     - `$id`: The ID parameter used to identify the specific folder associated with the file being stored.

2. Role Authorization:
   ```
   if (Auth::user()->role != 'admin')
      abort(403);
   ```
   - This code block checks if the authenticated user's role is not 'admin'.
   - If the condition is true, it aborts the request with a 403 HTTP response, indicating a "Forbidden" error.
   - This implies that only users with the 'admin' role are allowed to execute the code inside the function.

3. Request Validation:
   ```
   $request->validate([
      'file'  => ['required', 'mimes:pdf'],
      'title' => ['required', 'string'],
   ]);
   ```
   - This code block validates the data received through the `$request` object.
   - It ensures that the 'file' field is required and must be a PDF file (as specified by the 'mimes:pdf' rule).
   - Additionally, it validates that the 'title' field is required and must be a string.

4. Folder Retrieval:
   ```
   $folder = Folder::find($id);
   ```
   - This code retrieves the folder associated with the provided `$id` from the `Folder` model using the `find` method.
   - The retrieved folder is stored in the `$folder` variable for further use.

5. File Upload:
   ```
   $file = upload_file($request->file, 'file', 'qr_files');
   ```
   - This code calls a custom function named [upload_file](#upload_file-function) to handle the file upload process.
   - The function takes three parameters:
     1. `$request->file`: The uploaded file accessed from the `$request` object.
     2. `'file'`: The name of the file field in the request (in this case, it is 'file').
     3. `'qr_files'`: The directory or path where the uploaded file will be stored.

6. File Creation:
   ````
   File::create([
      'folder_id' => $folder->id,
      'file'      => $file,
      'title'     => $request->title,
   ]);
   ```
   - This code creates a new record in the `File` model (assuming there is a corresponding Eloquent model named `File`).
   - The `create` method is used to insert a new row into the associated database table.
   - The values for the 'folder_id', 'file', and 'title' columns are provided based on the retrieved folder, uploaded file, and the 'title' field from the request, respectively.

7. Response and Redirect:
   ````
   return back()->withStatus('File is created successfully');
   ```
   - This code redirects the user back to the previous page after the file has been successfully stored.
   - The `withStatus` method attaches a flash message to the redirect response, indicating that the file creation was successful.

[🔝 Back to contents](#contents)

### **upload_file-function**

`app\helpers.php`

```php
function upload_file($request_file, $prefix, $folder_name)
{
    $extension = $request_file->getClientOriginalExtension();
    $file_to_store = $prefix . '_' . time() . '.' . $extension;
    $request_file->storeAs('public/' . $folder_name, $file_to_store);
    return $folder_name.'/'.$file_to_store;
}
```
This function is responsible for handling the file upload process and returning the path of the stored file. It generates a unique file name using a prefix and the current timestamp, stores the file in the specified folder, and returns the file path.

1. Function Signature:
   ````php
   function upload_file($request_file, $prefix, $folder_name)
   ```
   - The `upload_file` function accepts three parameters:
     - `$request_file`: The uploaded file object obtained from the request.
     - `$prefix`: A string prefix used to generate a unique file name.
     - `$folder_name`: The name of the folder where the file will be stored.

2. File Extension Extraction:
   ````php
   $extension = $request_file->getClientOriginalExtension();
   ```
   - This code retrieves the original file extension of the uploaded file using the `getClientOriginalExtension` method.
   - The `$extension` variable stores the extracted file extension.

3. Generating Unique File Name:
   ````php
   $file_to_store = $prefix . '_' . time() . '.' . $extension;
   ```
   - This code generates a unique file name by concatenating the `$prefix`, current timestamp obtained using `time()`, the file extension, and underscores.
   - The resulting file name is stored in the `$file_to_store` variable.

4. Storing the File:
   ````php
   $request_file->storeAs('public/' . $folder_name, $file_to_store);
   ```
   - This code uses the `storeAs` method to store the uploaded file in a specific directory.
   - The first parameter `'public/' . $folder_name` represents the destination directory where the file will be stored. In this case, it uses the `public` disk and the specified `$folder_name` as the subdirectory.
   - The second parameter `$file_to_store` represents the desired file name for storing the file.

5. Returning the File Path:
   ````php
   return $folder_name.'/'.$file_to_store;
   ```
   - This code returns the file path as a string.
   - The returned path consists of the `$folder_name` and the `$file_to_store` concatenated with a forward slash (/) in between.

[🔝 Back to contents](#contents)

### **download_file-function**

`app\Http\Controllers\Dashboard\FolderFileController.php`

```php
public function download_file($id)
{
    $file = File::findOrFail($id);
    $path = storage_path('app/public/' . $file->file);
    $extension = pathinfo($file->file, PATHINFO_EXTENSION);
    return Response::download($path, $file->title . '.' . $extension);
}
```

This function retrieves a file record based on the provided ID, determines the file path and extension, and generates a download response with the appropriate file name. This allows users to download the file from the server.
## Step 1: Find the File
```
$file = File::findOrFail($id);
```
This line retrieves the file record from the database based on the provided `$id` using the `findOrFail` method. It assumes there is a model called `File` that represents the files in the system.

## Step 2: Define the File Path and Extension
```
$path = storage_path('app/public/' . $file->file);
$extension = pathinfo($file->file, PATHINFO_EXTENSION);
```
The function determines the file path by concatenating the storage path with the file's location stored in the `$file` object. It also extracts the file extension using the `pathinfo` function.

## Step 3: Download the File
```
return Response::download($path, $file->title . '.' . $extension);
```
The function generates a download response using the `Response::download` method. It takes the file path and the desired filename as arguments. The desired filename is constructed by combining the original file's title from the `$file` object with the extracted extension.


[🔝 Back to contents](#contents)
