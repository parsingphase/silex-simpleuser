Simple User Provider for Silex
==============================

[![Build Status](https://travis-ci.org/jasongrimes/silex-simpleuser.svg?branch=master)](https://travis-ci.org/jasongrimes/silex-simpleuser)
[![Total Downloads](https://poser.pugx.org/jasongrimes/silex-simpleuser/downloads.svg)](https://packagist.org/packages/jasongrimes/silex-simpleuser)
[![Latest Stable Version](https://poser.pugx.org/jasongrimes/silex-simpleuser/v/stable.svg)](https://packagist.org/packages/jasongrimes/silex-simpleuser)
[![Latest Unstable Version](https://poser.pugx.org/jasongrimes/silex-simpleuser/v/unstable.svg)](https://packagist.org/packages/jasongrimes/silex-simpleuser)

A simple database-backed user provider for use with the Silex [SecurityServiceProvider](http://silex.sensiolabs.org/doc/providers/security.html).

SimpleUser is intended to provide an easy way to set up user management (authentication, authorization, and user administration) in the Silex PHP micro-framework. It provides drop-in services for Silex that implement the missing user management pieces for the Security component. It includes a basic User model, a database-backed user manager, controllers and views for user administration, and various supporting features.


Demo
----

* [Online demo](http://silex-simpleuser-demo.grimesit.com/)
* [Demo source code](https://github.com/jasongrimes/silex-simpleuser-demo)


Quick start example config
--------------------------

This configuration should work out of the box to get you up and running quickly. See below for additional details.

Add this to your composer.json and then run `composer update`:

    "require": {
        "silex/silex": "~1.0",
        "symfony/twig-bridge": "~2.3",
        "jasongrimes/silex-simpleuser": "~1.0"
    }

Set up your Silex application something like this:


    <?php

    use Silex\Application;
    use Silex\Provider;

    //
    // Application setup
    //

    $app = new Application();
    $app->register(new Provider\DoctrineServiceProvider());
    $app->register(new Provider\SecurityServiceProvider());
    $app->register(new Provider\RememberMeServiceProvider());
    $app->register(new Provider\SessionServiceProvider());
    $app->register(new Provider\ServiceControllerServiceProvider());
    $app->register(new Provider\UrlGeneratorServiceProvider());
    $app->register(new Provider\TwigServiceProvider());

    // Register the SimpleUser service provider.
    $simpleUserProvider = new SimpleUser\UserServiceProvider();
    $app->register($simpleUserProvider);

    // ...

    //
    // Controllers
    //

    $app->mount('/user', $simpleUserProvider);

    /*
    $app->get('/', function () use ($app) {
        return $app['twig']->render('index.twig', array());
    });
    */

    // ...

    //
    // Configuration
    //

    $app['user.options'] = array();

    $app['security.firewalls'] = array(
        /* // Ensure that the login page is accessible to all, if you set anonymous => false below.
        'login' => array(
            'pattern' => '^/user/login$',
        ), */
        'secured_area' => array(
            'pattern' => '^.*$',
            'anonymous' => true,
            'remember_me' => array(),
            'form' => array(
                'login_path' => '/user/login',
                'check_path' => '/user/login_check',
            ),
            'logout' => array(
                'logout_path' => '/user/logout',
            ),
            'users' => $app->share(function($app) { return $app['user.manager']; }),
        ),
    );

    $app['db.options'] = array(
        'driver'   => 'pdo_mysql',
        'host' => 'localhost',
        'dbname' => 'mydbname',
        'user' => 'mydbuser',
        'password' => 'mydbpassword',
    );

    return $app;

Create the user database:

    mysql -uUSER -pPASSWORD MYDBNAME < vendor/jasongrimes/silex-simpleuser/sql/mysql.sql

You should now be able to create an account at the `/user/register` URL.
Make the new account an administrator by editing the record directly in the database and setting the `users.roles` column to `ROLE_USER,ROLE_ADMIN`.
(After you have one admin account, it can grant the admin role to others via the web interface.)


Config options
--------------

    $app['user.options'] = array(
        // Custom user class
        'userClass' => 'My\User',

        // Custom templates
        'layoutTemplate'   => 'layout.twig',
        'loginTemplate'    => 'login.twig',
        'registerTemplate' => 'register.twig',
        'viewTemplate'     => 'view.twig',
        'editTemplate'     => 'edit.twig',
        'listTemplate'     => 'list.twig',

        // Controller options
        'controllers' => array(
            'edit' => array(
                'customFields' => array('field' => 'Label'),
            ),
        ),
    );


More information
----------------

See the [Silex SimpleUser tutorial](http://www.jasongrimes.org/2014/09/simple-user-management-in-silex/).
