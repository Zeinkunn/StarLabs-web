# STARLABS WEBSITE

## Table of Contents

- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Usage](#usage)
- [Demo](#demo)
- [Documentation](#documentation)
- [Login](#login)
- [Register](#register)
- [Forgot Password](#forgot-password)
- [Reset Password](#reset-password)
- [User Profile](#user-profile)
- [Dashboard](#dashboard)
- [File Structure](#file-structure)
- [Licensing](#licensing)
- [Credits](#credits)

## Prerequisites

If you don't already have an Apache local environment with PHP and MySQL, use one of the following links:

- Windows: [Beginner's Guide to Setting Up Your Local Development Environment on Windows](https://updivision.com/blog/post/beginner-s-guide-to-setting-up-your-local-development-environment-on-windows)
- Linux & Mac: [Guide: What is LAMP and How to Install it on Ubuntu and macOS](https://updivision.com/blog/post/guide-what-is-lamp-and-how-to-install-it-on-ubuntu-and-macos)

Also, you will need to install [Composer](https://getcomposer.org/doc/00-intro.md) and [Laravel](https://laravel.com/docs/10.x).

## Installation

1. Unzip the downloaded archive.
2. Copy and paste the **soft-ui-dashboard-laravel-master** folder into your **projects** folder. Rename the folder to your project's name.
3. In your terminal, run `composer install`.
4. Copy `.env.example` to `.env` and update the configurations (mainly the database configuration).
5. In your terminal, run `php artisan key:generate`.
6. Run `php artisan migrate --seed` to create the database tables and seed the roles and users tables.
7. Run `php artisan storage:link` to create the storage symlink (if you are using **Vagrant** with **Homestead** for development, remember to ssh into your virtual machine and run the command from there).

## Usage

Register a user or login with the default user **admin@softui.com** and password **secret** from your database and start testing (make sure to run the migrations and seeders for these credentials to be available).

Besides the dashboard, the auth pages, the billing and table pages, there is also an edit profile page. All the necessary files are installed out of the box and all the needed routes are added to `routes/web.php`. Keep in mind that all of the features can be viewed once you login using the credentials provided or by registering your own user.

| HTML | Laravel |
| --- | --- |
| [![HTML](https://s3.amazonaws.com/creativetim_bucket/products/450/thumb/opt_sd_free_thumbnail.jpg)](https://www.creative-tim.com/product/soft-ui-dashboard) | [![Laravel](https://s3.amazonaws.com/creativetim_bucket/products/602/thumb/soft-ui-dashboard-laravel.jpg?1647531884)](https://www.creative-tim.com/product/soft-ui-dashboard-laravel) |

## Demo

| Register | Login | Dashboard |
| --- | --- | --- |
| [<img src="https://github.com/creativetimofficial/public-assets/blob/master/soft-ui-design-system-laravel/Register.png" width="322" />](https://soft-ui-dashboard-laravel.creative-tim.com/sign-up) | [<img src="https://github.com/creativetimofficial/public-assets/blob/master/soft-ui-design-system-laravel/Login.png?raw=true" width="322" />](https://soft-ui-dashboard-laravel.creative-tim.com/login) | [<img src="https://github.com/creativetimofficial/public-assets/blob/master/soft-ui-design-system-laravel/Dashboard.png?raw=true" width="322" />](https://soft-ui-dashboard-laravel.creative-tim.com/dashboard) |

| Forgot Password Page | Reset Password Page | Profile Page |
| --- | --- | --- |
| [<img src="https://github.com/creativetimofficial/public-assets/blob/master/soft-ui-design-system-laravel/Forgot-password.png" width="320" />](https://soft-ui-dashboard-laravel.creative-tim.com/login/forgot-password) | [<img src="https://github.com/creativetimofficial/public-assets/blob/master/soft-ui-design-system-laravel/Login.png" width="312" />](https://soft-ui-dashboard-laravel.creative-tim.com/) | [<img src="https://github.com/creativetimofficial/public-assets/blob/master/soft-ui-design-system-laravel/Profile.png" width="330" />](https://soft-ui-dashboard-laravel.creative-tim.com/laravel-user-profile) |

[View More](https://soft-ui-dashboard-laravel.creative-tim.com/dashboard)

## Documentation

The documentation for the Soft UI Dashboard Laravel is hosted at our [website](https://soft-ui-dashboard-laravel.creative-tim.com/documentation/getting-started/overview.html).

## Login

If you are not logged in, you can only access this page or the Sign Up page. The default URL takes you to the login page where you use the default credentials **admin@softui.com** with the password **secret**. Logging in is possible only with already existing credentials. For this to work, you should have run the migrations.

The `App\Http\Controllers\SessionController` handles the logging in of an existing user.

```php
public function store()
{
    $attributes = request()->validate([
        'email' => 'required|email',
        'password' => 'required'
    ]);

    if (Auth::attempt($attributes)) {
        session()->regenerate();
        return redirect('dashboard');
    } else {
        return back();
    }
}
```

## Register

You can register as a user by filling in the name, email, role, and password for your account. For your role, you can choose between Admin, Creator, and Member. It is important to know that an admin user has access to all the pages and actions, can delete, add, and edit other users, roles, items, tags, or categories; a creator user has access to category, tag, and item management but cannot add, edit, or delete other users; a member user has access to item management but cannot take any action. You can do this by accessing the sign-up page from the "**Sign Up**" button in the top navbar or by clicking the "**Sign Up**" button from the bottom of the login form. Another simple way is adding **/register** in the URL.

The `App\Http\Controllers\RegisterController` handles the registration of a new user.

```php
public function store()
{
    $attributes = request()->validate([
        'name' => ['required', 'max:50'],
        'email' => ['required', 'email', 'max:50', Rule::unique('users', 'email')],
        'password' => ['required', 'min:5', 'max:20'],
        'agreement' => ['accepted']
    ]);
    $attributes['password'] = bcrypt($attributes['password']);

    session()->flash('success', 'Your account has been created.');
    $user = User::create($attributes);
    Auth::login($user);
    return redirect('/dashboard');
}
```

## Forgot Password

If a user forgets the account's password, it is possible to reset the password. For this, the user should click on the "**here**" under the login form or add **/login/forgot-password** in the URL.

The `App\Http\Controllers\ResetController` takes care of sending an email to the user where they can reset the password afterward.

```php
public function sendEmail(Request $request)
{
    $request->validate(['email' => 'required|email']);

    $status = Password::sendResetLink(
        $request->only('email')
    );

    return $status === Password::RESET_LINK_SENT
        ? back()->with(['status' => __($status)])
        : back()->withErrors(['email' => __($status)]);
}
```

## Reset Password

The user who forgot the password gets an email on the account's email address. The user can access the reset password page by clicking the button found in the email. The link for resetting the password is available for 12 hours. The user must add the new password and confirm the password for their password to be updated. The user is redirected to the login page.

The `App\Http\Controllers\ChangePasswordController` helps the user reset the password.

```php
public function changePassword(Request $request)
{
    $request->validate([
        'token' => 'required',
        'email' => 'required|email',
        'password' => 'required|min:8|confirmed',
    ]);

    $status = Password::reset(
        $request->only('email', 'password', 'password_confirmation', 'token'),
        function ($user, $password) {
            $user->forceFill([
                'password' => Hash::make($password)
            ])->setRememberToken(Str::random(60));

            $user->save();

            event(new PasswordReset($user));
        }
    );

    return $status === Password::PASSWORD_RESET
        ? redirect('/login')->with('status', __($status))
        : back()->withErrors(['email' => [__($status)]]);
}
```

## User Profile

The profile can be accessed by a logged-in user by clicking "**User Profile**" from the sidebar or adding **/user-profile** in the URL. The user can add information like birthday, gender, phone number, location, language, or skills.

The `App\Http\Controllers\InfoUserController` handles the user's profile information.

```php
public function store(Request $request)
{
    $attributes = request()->validate([
        'name' => ['required', 'max:50'],
        'email' => ['required', 'email', 'max:50', Rule::unique('users')->ignore(Auth::user()->id)],
        'phone' => ['max:50'],
        'location' => ['max:70'],
        'about_me' => ['max:150'],
    ]);

    User::where('id', Auth::user()->id)
        ->update([
            'name' => $attributes['name'],
            'email' => $attributes['email'],
            'phone' => $attributes['phone'],
            'location' => $attributes['location'],
            'about_me' => $attributes['about_me'],
        ]);

    return redirect('/user-profile');
}
```

## Dashboard

You can access the dashboard either by using the "**Dashboard**" link in the left sidebar or by adding **/dashboard** in the URL after logging in.

## File Structure

```plaintext
app
├── Console
│   └── Kernel.php
├── Exceptions
│   └── Handler.php
├── Http
│   ├── Controllers
│   │   ├── ChangePasswordController.php
│   │   ├── Controller.php
│   │   ├── HomeController.php
│   │   ├── InfoUserController.php
│   │   ├── RegisterController.php
│   │   ├── ResetController.php
│   │   └── SessionController.php
│   ├── Kernel.php
│   └── Middleware
│       ├── Authenticate.php
│       ├── EncryptCookies.php
│       ├── PreventRequestsDuringMaintenance.php
│       ├── RedirectIfAuthenticated.php
│       ├── TrimStrings.php
│       ├── TrustHosts.php
│       ├── TrustProxies.php
│       └── VerifyCsrfToken.php
├── Models
│   └── User.php
├── Policies
│   └── UsersPolicy.php
├── Providers
│   ├── AppServiceProvider.php
│   ├── AuthServiceProvider.php
│   ├── BroadcastServiceProvider.php
│   ├── EventServiceProvider.php
│   └── RouteServiceProvider.php
config
├── app.php
├── auth.php
├── broadcasting.php
├── cache.php
├── cors.php
├── database.php
├── filesystems.php
├── hashing.php
├── logging.php
├── mail.php
├── queue.php
├── sanctum.php
├── services.php
├── session.php
├── view.php
database
├── factories
│   └── UserFactory.php
├── migrations
│   ├── 2014_10_12_000000_create_users_table.php
│   ├── 2014_10_12_100000_create_password_resets_table.php
│   ├── 2019_08_19_000000_create_failed_jobs_table.php
│   └── 2019_12_14_000001_create_personal_access_tokens_table.php
└── seeds
    ├── DatabaseSeeder.php
    └── UserSeeder.php
public
├── .htaccess
├── favicon.ico
├── index.php
├── css
│   ├── app.css
│   └── soft-ui-dashboard.css
├── js
│   └── app.js
├── assets
│   ├── demo.css
│   ├── docs-soft.css
│   └── docs.js
│   ├── css
│   │   ├── nucleo-icons.css
│   │   ├── nucleo-svg.css
│   │   ├── soft-ui-dashboard.css
│   │   ├── soft-ui-dashboard.css.map
│   │   └── soft-ui-dashboard.min.css
│   └── js
│       ├── soft-ui-dashboard.js
│       ├── soft-ui-dashboard.js.map
│       └── soft-ui-dashboard.min.js
│       ├── core
│           ├── bootstrap.bundle.min.js
│           ├── bootstrap.min.js
│           └── popper.min.js
resources
├── lang
│   └── en
│       ├── auth.php
│       ├── pagination.php
│       ├── passwords.php
│       └── validation.php
└── views
    ├── components
    │   └── fixed-plugins.blade.php
    ├── laravel-example
    │   ├── user-management.blade.php
    │   └── user-profile.blade.php
    ├── layouts
    │   ├── footers
    │   │   ├── auth
    │   │   │   └── footer.blade.php
    │   │   └── guest
    │   │       └── footer.blade.php
    │   ├── navbars
    │   │   ├── app.blade.php
    │   │   ├── auth
    │   │   │   ├── nav-rtl.blade.php
    │   │   │   ├── nav.blade.php
    │   │   │   ├── sidebar-rtl.blade.php
    │   │   │   └── sidebar.blade.php
    │   │   └── guest
    │   │       └── nav.blade.php
    │   ├── user_type
    │   │   ├── auth.blade.php
    │   │   └── guest.blade.php
    ├── session
    │   ├── login-session.blade.php
    │   ├── register.blade.php
    │   ├── reset-password
    │   │   ├── resetPassword.blade.php
    │   │   └── sendEmail.blade.php
    ├── billing.blade.php
    ├── dashboard.blade.php
    ├── profile.blade.php
    ├── rtl.blade.php
    ├── static-sign-in.blade.php
    ├── static-sign-up.blade.php
    ├── tables.blade.php
    └── virtual-reality.blade.php
routes
├── api.php
├── channels.php
├── console.php
└── web.php
```

## Licensing

- Copyright 2021 [Creative Tim](https://www.creative-tim.com?ref=readme-sudpl)
- Creative Tim [license](https://www.creative-tim.com/license?ref=readme-sudpl)

## Credits

- [Creative Tim](https://creative-tim.com/?ref=sudl-readme)
- [UPDIVISION](https://updivision.com)
