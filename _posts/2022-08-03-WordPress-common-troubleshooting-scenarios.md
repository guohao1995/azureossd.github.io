---
title: "WordPress Common Troubleshooting Scenarios"
author_name: "Arjun Baliga"
tags:
    - Azure App Service
    - WordPress
    - Mixed content error
    - Too many redirects
    - Debug WordPress
    - Troubleshooting
categories:
    - Azure App Service
    - WordPress
    - Mixed content error
    - Too many redirects
    - Debug WordPress
    - Troubleshooting
header:
    teaser: "/assets/images/azure-containerapps-logo.png" 
toc: true
toc_sticky: true
date: 2022-08-03 12:00:00
---
# Mixed Content error on WordPress site 
What does the mixed content error message indicate?

Normally when resources like images, CSS or JS files are requested with HTTPS and if they are loaded with HTTP protocol this can have an effect of mixed content error. 
 
You can confirm if your site is being affected by mixed content errors by checking the browser inspect element console. 
<br/><b>Error Message</b> - 
Mixed Content: The page URL was loaded over HTTPS but requested an insecure stylesheet.
 >![WordPress common troubleshooting scenarios](/media/2022/08/mixed-content-error.png)

Most browsers follow the RFC for Mixed Content, you can find information relevant on this restriction here:
<a>https://www.w3.org/TR/mixed-content/</a> and <a>https://tools.ietf.org/html/rfc6797</a>.

<b>How to resolve the mixed content error?<b>
1. In the wp-config.php, please check if the below configurations like WP_HOME and WP_SITEURL settings are set properly as per your requirement. Please refer to the screenshot below for the recommended settings.
if (isset($_SERVER['HTTP_X_FORWARDED_PROTO']) && $_SERVER['HTTP_X_FORWARDED_PROTO'] == 'https')
<br/>$_SERVER['HTTPS'] = 'on';
<br/>$http_protocol='http://';
<br/>if (!preg_match("/^localhost(:[0-9])*/", $_SERVER['HTTP_HOST']) && !preg_match("/^127\.0\.0\.1(:[0-9])*/", $_SERVER['HTTP_HOST'])) {
<br/>	$http_protocol='https://';
<br/>}
 >![WordPress common troubleshooting scenarios](/media/2022/08/wp_home_site_url.png)

 2.	In the Azure Portal, within the Custom Domains tab, check if the setting HTTPS only is turned on. 
 >![WordPress common troubleshooting scenarios](/media/2022/08/appservice_http_only.png)

 3. Purge the Wordpress cache from the admin Dashboard. 
  >![WordPress common troubleshooting scenarios](/media/2022/08/wp_admin_clear_cache.png)
 4. If you're still facing mixed content error search for HTTP references in the database and replace it with HTTPS. You can easily do it using the plugins below -
[Better Search and Replace](https://wordpress.org/plugins/better-search-replace/) or
[Search & Replace](https://wordpress.org/plugins/search-and-replace/)

5. Please install the WordPress plugin 'SSL Insecure Content Fixer' and select the appropriate HTTPS detection method as in the screenshot below. For example,HTTP_X_ARR_SSL setting works for Azure Linux App Service blessed images. 
 >![WordPress common troubleshooting scenarios](/media/2022/08/wp_detect_http_setting.png)

# How to Debug WordPress Errors
If your WordPress page is encountering application issues without any detailed errors, please use the below steps to debug further. 
  >![WordPress common troubleshooting scenarios](/media/2022/08/wp_critical_error.png)

1.	Edit the wp-config.php file and look for the following line of code:
```
define(‘WP_DEBUG’, false);
```
<br/> Please update the line to the changes below - 
```
define(‘WP_DEBUG’, true);
define(‘WP_DEBUG_LOG’, true);
define(‘WP_DEBUG_DISPLAY’, false);
@ini_set(‘display_errors’,0);
```
3.	Check `php-fpm.www.log` inside `/var/log/php-fpm/` and application logs for the detailed errors. 
4.	Once you fix the issue please revert the changes done in step 1.

Additionally, you can configure `WP_DEBUG_LOG` with a specific file path to save the log file in a different location. For more details, refer to [Debugging in WordPress – Advanced Administration Handbook | Developer.WordPress.org](https://developer.wordpress.org/advanced-administration/debug/debug-wordpress/#wp_debug_log). For example, you can write the log to `/home/site/wwwroot/wp-errors.log` with below configurations, this can be useful if you want to keep the log and access it through the file manager in Kudu at `https://<your-web-app-name>.scm.azurewebsites.net/newui/fileManager#`.

```
define( 'WP_DEBUG', true);
define( 'WP_DEBUG_LOG', '/home/site/wwwroot/wp-errors.log' );
define('WP_DEBUG_DISPLAY', false);
@ini_set('display_errors',0);
```

# How to resolve the 'Too Many Redirects' issue encountered on your WordPress site
1.	 Assess if the SSL, home_url and site_url configurations are set properly in your wp-config.php. Please refer to the screenshot and code below for the recommended settings.
<br/>if (isset($_SERVER['HTTP_X_FORWARDED_PROTO']) && $_SERVER['HTTP_X_FORWARDED_PROTO'] == 'https')
<br/>$_SERVER['HTTPS'] = 'on';
<br/>$http_protocol='http://';
<br/>if (!preg_match("/^localhost(:[0-9])*/", $_SERVER['HTTP_HOST']) && !preg_match("/^127\.0\.0\.1(:[0-9])*/", $_SERVER['HTTP_HOST'])) {
<br/>	$http_protocol='https://';
<br/>}
<br/>define('WP_HOME', $http_protocol . $_SERVER['HTTP_HOST']);
<br/>define('WP_SITEURL', $http_protocol . $_SERVER['HTTP_HOST']);
 >![WordPress common troubleshooting scenarios](/media/2022/08/wp_home_site_url.png)
2.	Disable all plugins by renaming the plugins folder to check if the redirection is being caused by any of the plugins. You can then re-enable them one by one to find the problematic plugin.
3.	Please check if the WordPress admin URL https://<site home url>/wp-admin or any other URLs within the admin page are also facing the 'too many redirects' issue. If only a few URLs are redirecting, it could be an incorrect redirect rule configured in the Redirect Plugins. Also, review the server configuration files like Nginx config files, .htaccess (in case you are using an Apache Web Server)  or web. config (in case you are using the IIS Web Server) for any redirect rules that may have been configured and modify/update the configurations as needed. 
