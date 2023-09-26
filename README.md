A date as a string is less reliable than an object instance, e.g. a Carbon-instance. It's recommended to pass Carbon objects between classes instead of date strings. Rendering should be done in the display layer (templates):
A date as a string is less reliable than an object instance, e.g. a Carbon-instance. It's recommended to pass Carbon objects between classes instead of date strings. Rendering should be done in the display layer (templates):
## Contents

[Authentication](#authentication)

[Create file functionality](#create-file-functionality)
[Create file modal](#create-file-modal)
[loadPdf function](#loadPdf-function)
[Store file function](#store-file-function)
  
[Folders index page](#folders)

[Files index page](#files)

### **authentication**

routes\api.php:

```php
Route::post('register',  [AuthController::class, 'register']);
Route::post('login'   ,  [AuthController::class, 'login']);
```

app\Http\Controllers\AuthController.php:

The constructor, this function is part of a controller class and is responsible for setting up themiddleware for authentication using Laravel Sanctum.
This middleware ensures that the user is authenticated using the Sanctum authentication guardbefore accessing the methods.
The $this->middleware('auth:sanctum')->only(['logout', 'user']); line specifies that the'auth:sanctum' middleware should be applied only to the 'logout' and 'user' methods.

```php
    public function __construct()
    {
        $this->middleware('auth:api')->only(['logout']);
    }
```

```php
    public function register (Request $request) 
    {
        $request->validate([
            'name'       => ['required', 'string'],
            'email'      => ['required', 'string', 'email', 'unique:users'],  
            'password'   => ['required', 'string', 'min:6', 'confirmed'],
        ]);

        $user = User::create([
            'name'      => $request->name,
            'email'     => $request->email,
            'password'  => Hash::make($request->password),
            'role'      => 'user',
            'balance'   => 0,
        ]);

        $token = $user->createToken('Proxy App')->accessToken;
        
        return response()->json([
            'user' => new UserResource($user),
            'token' => $token,
        ], 200);
    }
```

```php
    public function login (Request $request) 
    {
       $request->validate([
            'email'      => ['required', 'string', 'email'],  
            'password'   => ['required', 'string'],
        ]);

        $user = User::where('email', $request->email)->first();

        if ($user) {
            if (Hash::check($request->password, $user->password)) {
                $token = $user->createToken('Mega Panel App')->accessToken;

                return response()->json([
                    'user' => new UserResource($user),
                    'token' => $token,
                ], 200);   
            }
        }
        
        return response()->json([
            'message' => 'email or password is incorrect.',
            'errors' => [
                'email' => ['email or password is incorrect.']
            ]
        ], 422);
    }
```

[🔝 Back to contents](#contents)

### **create-file-functionality**

[🔝 Back to contents](#contents)

### **create-file-modal**

resources\views\dashboard\file_view.blade.php:

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

app\Http\Controllers\Dashboard\FolderFileController.php:

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

app\helpers.php:

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

### **folders**

resources\views\dashboard\folder_view.blade.php:

[🔝 Back to contents](#contents)

### **files**

resources\views\dashboard\file_view.blade.php:

```html
<x-layouts.dashboard>
    <x-slot name="page">users.view</x-slot>
    <x-slot name="title">{{ $user }} / {{ $title }} {{ __('custom.Files') }}</x-slot>
    <x-slot name="card_title">{{ $user }} / {{ $title }} {{ __('custom.Files') }}</x-slot>
    <x-slot name="script">
        <script>
            $(document).ready(function() {
                $('#table').DataTable();
            });
        </script>
    </x-slot>
    <x-slot name="head">
        <SCript>
            var loadPdf = function(event) {
                var name = event.target.files[0].name;
                name = name.substr(0, name.lastIndexOf('.'));
                document.getElementById('pdf-title').value = name;
            }
        </SCript>

    </x-slot>
    <STYle>
        label.custom-file-button {
            position: relative;
        }

        label.custom-file-button:before {
            content: "📷 Take or upload picture";
            position: absolute;
            left: 0;
            padding: 10px;
            background: #1b87d5;
            color: #fff;
            width: auto;
            text-align: center;
            border-radius: 5px;
            cursor: pointer;
        }

        label.custom-file-button:hover:before {
            background: #146bac;
        }

        .custom-file-input {
            visibility: hidden;
        }
    </STYle>
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

    <div class="table-responsive">
        <table class="table" id="table">
            <thead class=" text-primary">
                <tr>
                    <th class="text-center">{{ __('custom.File Title') }}</th>
                    <th class="text-center">{{ __('custom.Creation Date') }}</th>
                    <th class="text-center th-action">{{ __('custom.Actions') }}</th>
                </tr>
            </thead>
            <tbody>
                @foreach ($files as $file)
                    <tr>
                        <td class="text-center"><img src="/img/pdf-icon/pdf-icon.png"
                                style="height:30px;width:30px;object-fit: cover;">
                            {{ $file->title }}</td>
                        <td class="text-center">{{ $file->created_at->diffForHumans() }}</td>
                        <td class="td-actions text-center d-flex justify-content-center">
                            <button type="button" rel="tooltip" class="btn btn-success btn-round" data-toggle="modal"
                                data-target="#editModal{{ $file->id }}">
                                <i class="material-icons" style=" display: contents;">edit</i>
                            </button>
                            {{-- Edit Modal --}}
                            <form method="post"
                                action="{{ route('folder_files.files.update', [$file->id, $file->folder_id]) }}"
                                autocomplete="off" class="form-horizontal" enctype="multipart/form-data">
                                @csrf
                                @method('PUT')
                                <div class="modal fade" id="editModal{{ $file->id }}" tabindex="-1"
                                    role="dialog" aria-labelledby="editModalTitle" aria-hidden="true">
                                    <div class="modal-dialog modal-dialog-centered" role="document">
                                        <div class="modal-content">
                                            <div class="modal-header">
                                                <h5 class="modal-title" id="editModalLabel">
                                                    {{ __('custom.Edit file title') }}</h5>
                                                <button type="button" class="close" data-dismiss="modal"
                                                    aria-label="Close">
                                                    <span aria-hidden="true">&times;</span>
                                                </button>
                                            </div>
                                            <div class="modal-body">
                                                <div id="add-error" class="alert alert-danger"
                                                    style="color: red;display:none;">
                                                    {{ __('custom.The entered values ​​are not valid') }}
                                                </div>
                                                <div class="row">
                                                    <label class="col-sm-3 col-form-label"
                                                        for="input-title-en">{{ __('custom.Title') }}</label>
                                                    <div class="col-sm-8">
                                                        <div
                                                            class="form-group{{ $errors->has('title') ? ' has-danger' : '' }}">
                                                            <input
                                                                class="form-control{{ $errors->has('title') ? ' is-invalid' : '' }}"
                                                                name="title" id="input-title-en" type="text"
                                                                placeholder="{{ __('custom.Enter Title') }}"
                                                                value="{{ $file->title }}" aria-required="true" />
                                                            @if ($errors->has('title'))
                                                                <span id="title-en-error" class="error text-danger"
                                                                    for="input-title-en">{{ $errors->first('title') }}</span>
                                                            @endif
                                                        </div>
                                                    </div>
                                                </div>
                                                <div class="modal-footer" style="border-top:none;">
                                                    <button type="submit"
                                                        class="btn btn-success">{{ __('custom.Save') }}</button>
                                                </div>
                                            </div>
                                        </div>
                                    </div>
                                </div>
                            </form>
                            {{-- End Modal --}}
                            <button type="button" data-toggle="modal"
                                data-target="#exampleModal{{ $file->id }}" style="margin-inline: 6px;"
                                rel="tooltip" class="btn btn-danger btn-round">
                                <i class="material-icons" style=" display: contents;">close</i>
                            </button>
                            <form class="delete"
                                action="{{ route('folder_files.files.destroy', [$file->id, $file->folder_id]) }}"
                                method="POST">
                                @csrf
                                @method('DELETE')
                                <!-- Modal -->
                                <div class="modal fade" id="exampleModal{{ $file->id }}" tabindex="-1"
                                    role="dialog" aria-labelledby="exampleModalLabel" aria-hidden="true">
                                    <div class="modal-dialog" role="document">
                                        <div class="modal-content">
                                            <div class="modal-header">
                                                <h5 class="modal-title" id="exampleModalLabel">Hi @if (Auth::user()->name)
                                                        {{ Auth::user()->name }}
                                                    @endif !!! </h5>
                                                <button type="button" class="close" data-dismiss="modal"
                                                    aria-label="Close">
                                                    <span aria-hidden="true">&times;</span>
                                                </button>
                                            </div>
                                            <center>
                                                <div class="modal-body">
                                                    {{ __('custom.Are you sure you want to delete this file?') }}
                                                </div>
                                            </center>
                                            <div class="modal-footer">
                                                <button type="button" id="close" class="btn btn-secondary"
                                                    style=" right: 30px;" data-dismiss="modal">Cancel</button>
                                                <button type="submit" class="btn btn-danger">Delete</button>
                                            </div>
                                        </div>
                                    </div>
                                </div>
                            </form>
                            &nbsp;
                            <a href="{{ asset('storage'.DIRECTORY_SEPARATOR. $file->file) }} " target="_blank">
                            <button type="button" rel="tooltip" class="btn btn-primary btn-round">
                            <i class="material-icons" style=" display: contents;">download</i>
                        </button>
                    </a>
                        </td>
                    </tr>
                @endforeach
            </tbody>
        </table>
    </div>
</x-layouts.dashboard>

```

[🔝 Back to contents](#contents)
