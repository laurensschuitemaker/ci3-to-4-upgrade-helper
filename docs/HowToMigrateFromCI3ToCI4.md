# How to Migrate from CI3 to CI4

## Install CodeIgniter4

See <https://codeigniter4.github.io/CodeIgniter4/installation/index.html>.

**Note:** CodeIgniter 4.0.4 is not supported. Use 4.0.5-dev (`develop` branch) or later.

## Install ci3-to-4-migration-helper

```
$ composer require kenjis/ci3-to-4-migration-helper:1.x-dev
```

## Config

### config.php

1. Migrate `application/config/config.php` to `app/Config/App.php` manually. You can set your own values like base_url with `.env` file.

### Routing

1. Migrate `application/config/routes.php` to `app/Config/Routes.php` manually.

See <https://codeigniter4.github.io/CodeIgniter4/incoming/routing.html#setting-your-own-routing-rules>.

### Custom Config Files

1. Convert custom config files to Config classes manually.

See <https://codeigniter4.github.io/CodeIgniter4/general/configuration.html#creating-configuration-files>.

### Other Config Files

1. Migrate remaining `application/config/*.php` to `app/Config/*.php` manually. You can set your own values like database password with `.env` file.

### app/Config/View.php

1. If you use `$this->config` in view files, you need to add the following code.

```diff
--- a/app/Config/View.php
+++ b/app/Config/View.php
@@ -3,9 +3,12 @@
 namespace Config;
 
 use CodeIgniter\Config\View as BaseView;
+use Kenjis\CI3Compatible\Traits\View\ThisConfigInView;
 
 class View extends BaseView
 {
+   use ThisConfigInView;
+
    /**
     * When false, the view method will clear the data between each
     * call. This keeps your data safe and ensures there is no accidental
```

## Hooks

1. Migrate `application/config/hooks.php` to `app/Config/Events.php` or `app/Config/Filters.php` manually.
2. Migrate `application/hooks/*` to *Events* or *Controller Filters* manually.

See <https://codeigniter4.github.io/CodeIgniter4/extending/events.html> or <https://codeigniter4.github.io/CodeIgniter4/incoming/filters.html>.

## Controllers

### Copy Controller files

1. Copy `application/controllers/*` to `app/Controllers/*`.
2. Rename the sub-folder names so that only the first letter is uppercase.

### Add Namespace and Use statement

1. Add `namespace App\Controllers;`.
2. Add `use Kenjis\CI3Compatible\Core\CI_Controller;`

Example:
```php
namespace App\Controllers; // Add

use Kenjis\CI3Compatible\Core\CI_Controller; // Add

class News extends CI_Controller
{
    ...
}
```

## Models

### Copy Model files

1. Copy `application/models/*` to `app/Models/*`.
2. Rename the sub-folder names so that only the first letter is uppercase.

### Add Namespace and Use statement

1. Add `namespace App\Models;`.
2. Add `use Kenjis\CI3Compatible\Core\CI_Model;`

Example:
```php
namespace App\Models; // Add

use Kenjis\CI3Compatible\Core\CI_Model; // Add

class News_model extends CI_Model
{
   ...
}
```

## Libraries

### Copy Library files

1. Copy `application/libraries/*` to `app/Libraries/*`.
2. Rename the sub-folder names so that only the first letter is uppercase.

### Add Namespace

1. Add `namespace App\Libraries;`.

Example:
```php
namespace App\Libraries; // Add

class Seeder
{
   ...
}
```

### Form_validation

1. CI4 has View templates to display errors.
- The CI3 methods to customize error output are not supported.
- Create your own templates, and configure it in `app/Config/Validation.php`.
- See <https://codeigniter4.github.io/CodeIgniter4/libraries/validation.html#customizing-error-display>.
2. CI4 has no [Callbacks](https://codeigniter.com/userguide3/libraries/form_validation.html#callbacks-your-own-validation-methods) nor [Callable](https://codeigniter.com/userguide3/libraries/form_validation.html#callable-use-anything-as-a-rule).
- Create your own Rule classes, and configure it in `app/Config/Validation.php`.
- See <https://codeigniter4.github.io/CodeIgniter4/libraries/validation.html#creating-custom-rules>.
3. `set_message()`
- If you create a custom rule, use the second param `&$error` and set the error message. See <https://codeigniter4.github.io/CodeIgniter4/libraries/validation.html#creating-custom-rules>.
- Otherwise use `setError(string $field, string $error)`.

### Pagination

1. CI4 has View templates for Pagination.
- The CI3 configurations to customize pagination links are not supported.
- Create your own templates, and configure it in `app/Config/Pager.php`.
- Use `$pager->hasNextPage()` and `$pager->getNextPage()` instead of `$pager->hasNext()` `$pager->getNext()` for the next page link.
- A sample file is included in `src/CI3Compatible/Views/Pager/`. You could use it.

*app/Config/Pager.php*
```php
    public $templates = [
        'default_full'   => 'Kenjis\CI3Compatible\Views\Pager\default_full',
        'default_simple' => 'CodeIgniter\Pager\Views\default_simple',
        'default_head'   => 'CodeIgniter\Pager\Views\default_head',
    ];
```

- See <https://codeigniter4.github.io/CodeIgniter4/libraries/pagination.html#customizing-the-links>.
2. CI4 uses the actual page number only. You can't use the starting index (offset) for the items which is the default in CI3. So if you use *offset*, you have to convert *page* to *offset*.

```php
$offset = max(($page - 1), 0) * $per_page;
```

3. CI4 gets the base URL automatically from the current URL. You can't set it by config.

## Views

### Copy View files

1. Copy `application/views/*` to `app/Views/*`.

## Helpers

### Copy Helper files

1. Copy `application/helper/*` to `app/Helpers/*`.

### URL Helper

#### redirect()

1. CI4 changed `redirect()` API.
- See <https://codeigniter4.github.io/CodeIgniter4/general/common_functions.html#redirect>.
- Replace `redirect($uri)` with `return redirect()->to(site_url($uri))`, when you can return Response object.
- Replace it with`throw new \CodeIgniter\Router\Exceptions\RedirectException($uri)`, when you cannot return Response object.
- Or you could use `redirect_()` that *ci3-to-4-migration-helper* provides.

#### base_url()

1. CI4's `base_url()` removes the last `/`.
- If you don't want it, use `base_url_()` that *ci3-to-4-migration-helper* provides after `$this->load->helper('url')`.