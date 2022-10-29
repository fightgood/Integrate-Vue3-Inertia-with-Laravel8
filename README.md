# Integrate-Vue3-Inertia-with-Laravel8
> **NOTICE:** This guide was written using Laravel 8. It also works with Laravel 9 that's using Laravel Mix. If you're using Laravel 9 with Vite, follow this tutorial.

Before we dive in deep, we just want to make sure we have all the tools we need. We'll be using PHP 8, so make sure you have that installed, Composer and NPM. I'll briefly go over how to install Composer and NPM.

# 1. Installing Laravel
-----------------------------------------

Please note that I'm using Laravel 8 and PHP 7.4.32.

Making sure we are in the desired folder we're going to require the Laravel's installer globally and then use it to create a new app called "**laravel8-vue3-inertia**" (this will automatically create the folder with the same name).

### 1.1 Create Project

*Using this command:*

    composer create-project laravel/laravel laravel8-vue3-inertia

### 1.2 Change Directory

*Using this command:*

    cd laravel8-vue3-inertia

### 1.3 Install: npm
*Using this command:*

    npm install
and

*Using this command:*  

    npm install --save @types/node

### 1.4 Run

*Using this command:*

    php artisan serv

### 1.4 Result
*The result like this:*

![](https://i.ibb.co/wWNGdm4/5-Result.png)

# 2. Install Vue.js
--------------------------------------

### 2.1 Install: vue
We'll be using version 3 of Vue. Let's add Vue 3.  

*Using this command:*

    npm install vue@next

### 2.2 Run

*Using this command:*

    npm run dev

# 3. Installing Inertia
---------------------------------------------------
Inertia is a new approach to building classic server-driven web apps. We call it the modern monolith.

Inertia allows you to create fully client-side rendered, single-page apps, without much of the complexity that comes with modern SPAs. It does this by leveraging existing server-side frameworks.

Inertia has no client-side routing, nor does it require an API. Simply build controllers and page views like you've always done!

### 3.1 Server-side setup

The first step when installing Inertia is to configure your server-side framework. Inertia ships with official server-side adapters for Laravel and Rails.

##### Install dependencies

Install the Inertia server-side adapters using the preferred package manager for that language or framework.

*Using this command:*

    composer require inertiajs/inertia-laravel
 
##### Middleware

Next, setup the Inertia middleware. In the Rails adapter, this is configured automatically for you. However, in Laravel you need to publish the `HandleInertiaRequests` middleware to your application, which can be done using this artisan command:

*Using this command:*

    php artisan inertia:middleware

Once generated, register the `HandleInertiaRequests` middleware in your `App\Http\Kernel.php`, as the last item in your `web` middleware group.

    'web' => [
        // ...
        \App\Http\Middleware\HandleInertiaRequests::class,
    ],

This middleware provides a version() method for setting your asset version, and a share() method for setting shared data. Please see those pages for more information.

### 3.2 Client-side setup

Once you have your server-side framework configured, you then need to setup your client-side framework. Inertia currently provides support for React, Vue, and Svelte.

##### Install dependencies

Install the Inertia client-side adapters using NPM command.

*Using this command:*

    npm install @inertiajs/inertia @inertiajs/inertia-vue

##### Progress indicator

Since Inertia requests are made via XHR, there's no default browser loading indicator when navigating from one page to another. To solve this, Inertia provides an optional progress library, which shows a loading bar whenever you make an Inertia visit. To use it, start by installing it.

*Using this command:*

    npm install @inertiajs/progress

# 4. Install Ziggy
--------------------------------------

Ziggy provides a JavaScript `route()` helper function that works like Laravel's, making it easy to use your Laravel named routes in JavaScript.

Install Ziggy into your Laravel app with .

    composer require tightenco/ziggy

Add the `@routes` Blade directive to your main layout (_before_ your application's JavaScript), and the `route()` helper function will now be available globally!

> By default, the output of the `@routes` Blade directive includes a list of all your application's routes and their parameters. This route list is included in the HTML of the page and can be viewed by end users. To configure which routes are included in this list, or to show and hide different routes on different pages, see Filtering Routes.

# 5. Connect everything together
--------------------------------------

Now we have everything installed and ready to be used. We have installed **Laravel 8, Vue 3, Inertia** and **Ziggy**.

### 5.1 Setup - app.blade.php

Let's start by setting up our one and only **blade** template. We're going to rename the `welcome.blade.php` to `app.blade.php` inside `resources/views`. We're also going to remove all its content and replace it with the following:  

    <!DOCTYPE html>
    <html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
    
        <head>
            <meta charset="utf-8">
            <meta name="viewport" content="width=device-width, initial-scale=1">
    
            @routes
            <link href="{{ asset(mix('css/app.css')) }}" rel="stylesheet">
            <script src="{{ asset(mix('js/app.js')) }}" defer></script>
            <script src="{{ asset(mix('js/manifest.js')) }}" defer></script>
            <script src="{{ asset(mix('js/vendor.js')) }}" defer></script>
            <title>Integrate-Vue3-Inertia-with-Laravel8</title>
            @inertiaHead
        </head>
    
        <body>
            @inertia
        </body>
    
    </html>

So first of all you will notice we don't have any `<title>`. This is because we need it to be dynamic and we can set that using Inertia's `<Head>` component. That's why you can see that we've also added the `@inertiaHead` directive.

We have added the `@routes` directive to pass the Laravel's routes in the document's `<head>`.

We are importing our `app.css` and also a bunch of `.js` we are going to take care shortly.

In the `<body>` we only use the `@inertia` directive which renders a `div` element with a bunch of data passed to it using a `data-page` attribute.

### 5.2 Ziggy - Setup

Let's get back to Ziggy and generate the `.js` file that contains all of our routes. We'll gonna import this into our `app.js` a bit later.  

    php artisan ziggy:generate resources/js/ziggy.js

To resolve `ziggy` in Vue, we'll have to add an alias to the Vue driver in `webpack.mix.js`:  

    const path = require("path");
    
    // Rezolve Ziggy
    mix.alias({
        ziggy: path.resolve("vendor/tightenco/ziggy/dist/vue"),
    });

### 5.3 Setup - app.js

Let's move on by setting up our app.js file. This is our main main file we're going to load in our blade template.

Now open `resources/js/app.js` and delete everything from it and add the following chunk of code:  

    import { createApp, h } from "vue";
    import { createInertiaApp, Link, Head } from "@inertiajs/inertia-vue3";
    import { InertiaProgress } from "@inertiajs/progress";
    
    import { ZiggyVue } from "ziggy";
    import { Ziggy } from "./ziggy";
    
    InertiaProgress.init();
    
    createInertiaApp({
        resolve: async (name) => {
            return (await import(`./Pages/${name}`)).default;
        },
        setup({ el, App, props, plugin }) {
            createApp({ render: () => h(App, props) })
                .use(plugin)
                .use(ZiggyVue, Ziggy)
                .component("Link", Link)
                .component("Head", Head)
                .mixin({ methods: { route } })
                .mount(el);
        },
    });

What does this is to import Vue, Inertia, Inertia Progress and Ziggy and then create the Inertia App. We're also passing the `Link` and `Head` components as globals because we're going to use them a lot.

### 5.4 Setup - Folders & Files

Inertia will load our pages from the `Pages` directory so I'm gonna create 3 demo pages in that folder (`About.vue`, `Contact.vue`,  `Home.vue`). Like so:

![][image_ref_9ke1v1zd]

Each page will container the following template. The `Homepage` text will be replaced based on the file's name:  

    <template>
        <h1>Homepage</h1>
    </template> 

### 5.5 Setup - webpack.mix.js

The next step is to add the missing pieces to the `webpack.mix.js` file. Everything needs to look like this:

    const path = require("path");
    const mix = require("laravel-mix");
    
    // Rezolve Ziggy
    mix.alias({
        ziggy: path.resolve("vendor/tightenco/ziggy/dist/vue"),
    });
    
    // Build files
    mix.js("resources/js/app.js", "public/js")
        .vue({version: 3})
        .webpackConfig({
            resolve: {
                alias: {
                    "@": path.resolve(__dirname, "resources/js"),
                },
            },
        })
        .extract()
        .postCss("resources/css/app.css", "public/css", [//
        ])
        .version();


You can see that we're specifying the Vue version that we're using, we're also setting and alias (`@`) for our root js path and we're also using `.extract()` to split our code into smaller chunks (optional, but better for production in some use cases).

### 5.6 Settup - web.php

We've taken care of almost everything. Not we just need to create routes for each of the Vue pages we have created.

Let's open the `routes/web.php` file and replace everything there with the following:  

    <?php
    
    use Illuminate\Support\Facades\Route;
    use Inertia\Inertia;
    
    Route::get(
        '/',
        static function () {
            return Inertia::render(
                'Home',
                [
                    'title' => 'Homepage',
                ]
            );
        }
    )->name('homepage');
    
    Route::get(
        '/about',
        static function () {
            return Inertia::render(
                'About',
                [
                    'title' => 'About',
                ]
            );
        }
    )->name('about');
    
    Route::get(
        '/contact',
        static function () {
            return Inertia::render(
                'Contact',
                [
                    'title' => 'Contact',
                ]
            );
        }
    )->name('contact');


You can notice right away that we're not returning any traditional blade view. Instead we return an `Inertia::render()` response which takes 2 parameters. The first parameter is the name of our Vue page and the 2nd is an array of properties that will be passed to the Vue page using `$page.props`.

### 5.7 Modifying the Vue pages

Knowing this we can modify our pages to the following template and also add a navigation to them:  

    <template>
    
        <Head>
            <title>{{ $page.props.title }} - Hello World</title>
        </Head>
    
        <body>
            <span>
                <a :href="route('homepage')"><button>Homepage</button></a>
                <a :href="route('about')"><button>About </button></a>
                <a :href="route('contact')"><button>Contact </button></a>
            </span>
    
            <div>
                <h1>This is: {{ $page.props.title }}</h1>
            </div>
        </body>
    </template>

Now we have a simple navigation on each page and also a dynamic page `<title>`.

# 6. Testing
--------------------------------------

The only thing left now is to compile everything and start the server:  

    npm install
    npm update vue-loader
    npm run dev
    npm install vue-loader@^16.2.0 --save-dev --legacy-peer-deps
    npm run dev
    php artisan serve
   
Result:

![][image_ref_3gdqpb4e]


[image_ref_9ke1v1zd]: data:image/png;base64,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


[image_ref_3gdqpb4e]: data:image/png;base64,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
    