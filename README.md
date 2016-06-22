<!-- .slide: data-background-color="#262361" -->
# Migrate Shotgun Tour
Ryan Weal & Novella Chiechi

Kafei Interactive Inc.

2016-06-17 Montréal

Formatted as slides here:
https://kafeiinteractif.github.io/shotgun-migrate-tour/#/




<!-- .slide: data-background-color="#262361" -->
## What is migrate?

 - We are going to install Drupal 8
 - Connect to (a copy?) of the old site
 - Install migration modules
 - Auto or manually configure the migraitons
 - Sources, Processors, Destinations (plugins)
 - Run the migration! Import content!



<!-- .slide: data-background-color="#262361" -->
## Historical Note
###Forget what you know about migrate for Drupal 7

 - Creates content types
 - Imports things now in core
 - Tries to preserve settings
 - Turnkey solution for some users



<!-- .slide: data-background-color="#262361" -->
## modules
core
(8.1.x and up)
  - migrate
  - migrate_drupal
  - migrate_drupal_ui

contrib
(8.2.x branches and up only!)
  - migrate_plus
  - migrate_tools
  - migrate_upgrade

shell
(a recent version supporting Drupal 8)
  - drush (drupal console also has some things)




<!-- .slide: data-background-color="#262361" -->
## modules : other sources

contrib
  - migrate_source_csv
  - migrate_source_json
  - xml (part of migrate_plus)
  - wordpress_migrate



<!-- .slide: data-background-color="#262361" -->
## Migration approaches

<table border="1" cellpadding="1" cellspacing="1" style="width: 100%;">
  <tbody>
    <tr>
      <td>Automatic</td>
      <td>Hybrid</td>
      <td>Manual</td>
    </tr>
    <tr>
      <td>
      <p>Let Drupal detect whatever it can migrate, run all the possible migrations.</p>
      <ul>
         <li>migrate</li>
         <li>migrate_drupal</li>
         <li>migrate_drupal_ui</li>
       </ul>
      </td>
      <td><p>Let Drupal detect what it can migrate, and then modify that configuration before running it.</p>

      <ul>
         <li>migrate_upgrade (contrib)</li>
      </ul>

      <p>+automatic +manual</p>
      </td>
      <td><p>Construct your own migrations, defined in configuration-like files.</p>
      <ul>
         <li>migrate_plus</li>
         <li>migrate_tools</li>
         <li>drush</li>
      </ul>

      <p>+automatic</p>
      </td>
    </tr>
  </tbody>
</table>




<!-- .slide: data-background-color="#262361" -->
## Automatic Method

<blockquote>As a goat, I don't want to do any configuration or coding.  I just
want my content from Drupal 6 (or 7) and I want it now. Why would I want to install Drush?</blockquote>

At the end of this section you will know how to run migrations using the UI.



<!-- .slide: data-background-color="#262361" -->
## Automatic Method Setup

 - Modules: migrate, migrate_drupal, migrate_drupal_ui
 - Visit /upgrade after enabling these modules
 - Input DB and file paths
 - Run it.

<img src='images/figure4.png'>



<!-- .slide: data-background-color="#262361" -->
Import success!

Add D7 hash to settings.php!

<img src='images/figure8.png'>



<!-- .slide: data-background-color="#262361" -->
## Automatic Method Extras

Possible to pre-fill database values in UI using hook_form_alter:

    /**
     * Implements hook_form_alter().
     */
    function migrate_presets_form_alter(&$form, \Drupal\Core\Form\FormStateInterface $form_state, $form_id) {
      if ($form_id == 'migrate_drupal_ui_form') {
        $form['database']['settings']['mysql']['host']['#default_value'] = 'db';
        $form['database']['settings']['mysql']['database']['#default_value'] = 'cooks_legacy';
        $form['database']['settings']['mysql']['username']['#default_value'] = 'root';
        $form['database']['settings']['mysql']['password']['#value'] = 'my-secret-pw';
        $form['database']['settings']['mysql']['advanced_options']['port']['#default_value'] = '3306';
        $form['source']['source_base_path']['#default_value'] = '/path/to/files';
      }

There is also a settings.php hack but it does not work for UI.



<!-- .slide: data-background-color="#262361" -->
## Automatic Method: Further reading

 - Performing Drupal Content Migrations on Pantheon
   https://pantheon.io/blog/performing-drupal-content-migrations-pantheon



<!-- .slide: data-background-color="#718522" -->
## Hybrid Method

<blockquote> As a goat, I know writing code from scatch can take a long time,
but migrate can detect all of my configuration so I can skip that. I only want
to run part of it. Or I want to change it just a bit.</blockquote>

At the end of this section you will have working knowledge of how to use drush
for migrations.



<!-- .slide: data-background-color="#718522" -->
## Hybrid Method Setup
Migrating from Drupal 6 or 7

 - Enable: migrate, migrate_drupal, migrate_drupal_ui, migrate_tools, migrate_plus, migrate_upgrade
 - Install drush
 - Add 'upgrade' database to settings.php if you don't want to put on command line each time:

     <pre><code>$databases['upgrade']['default'] = array (
      'database' => 'cooks_legacy',
      'username' => 'root',
      'password' => 'my-secret-pw',
      'prefix' => '', 
      'host' => 'db',
      'port' => '3306',
      'namespace' => 'Drupal\Core\Database\Driver\mysql',
      'driver' => 'mysql',
    );</code></pre>

The 'upgrade' database key is only used with migrate_upgrade module.



<!-- .slide: data-background-color="#718522" -->
## Hybrid Method Discovery

 - load the config, only run a few? or,
 - export the config
 - modify the config
 - import the new config
 - run the config



<!-- .slide: data-background-color="#718522" -->
## Hybrid Workflow: Export

Configure the Migrations

    drush migrate-upgrade \
    --legacy-db-url=mysql://user:password@server/db \
    --legacy-root=http://example.com \
    --configure-only

Remember if you created 'upgrade' db in settings.php you can skip the --legacy-db-url.

     # drush migrate-status                                                    
     Group: migrate_drupal_6                         Status  Total  Imported  Unprocessed  Last imported 
     upgrade_block_content_type                      Idle    1      0         1                          
     upgrade_contact_category                        Idle    0      0         0                          
     upgrade_d6_date_formats                         Idle    0      0         0                          
     upgrade_d6_dblog_settings                       Idle    0      0         0                          
     upgrade_d6_file_settings                        Idle    0      0         0                          
     upgrade_d6_imagecache_presets                   Idle    3      0         3                          
     upgrade_d6_search_settings                      Idle    0      0         0                          
     upgrade_d6_system_cron                          Idle    0      0         0                          
     upgrade_d6_system_date                          Idle    1      0         1                          
     upgrade_d6_system_file                          Idle    1      0         1                          
     upgrade_d6_system_image                         Idle    1      0         1                          
     upgrade_d6_system_image_gd                      Idle    0      0         0                          
     upgrade_d6_system_logging                       Idle    0      0         0                          
     upgrade_d6_system_maintenance                   Idle    0      0         0                          
     upgrade_d6_system_performance                   Idle    0      0         0                          
     upgrade_d6_system_rss                           Idle    0      0         0                          
     upgrade_d6_system_site                          Idle    1      0         1                          
     upgrade_d6_url_alias                            Idle    0      0         0                          
     upgrade_d6_user_mail                            Idle    0      0         0                          
     upgrade_d6_user_settings                        Idle    1      0         1                          
     upgrade_menu_settings                           Idle    0      0         0                          
     upgrade_search_page                             Idle    0      0         0                          
     upgrade_taxonomy_settings                       Idle    0      0         0                          
     upgrade_text_settings                           Idle    0      0         0                          
     upgrade_update_settings                         Idle    0      0         0                          
     upgrade_block_content_body_field                Idle    1      0         1                          
     upgrade_d6_contact_settings                     Idle    0      0         0                          
     upgrade_menu                                    Idle    4      0         4                          
     upgrade_d6_filter_format                        Idle    2      0         2                          
     upgrade_d6_custom_block                         Idle    0      0         0                          
     upgrade_d6_user_role                            Idle    2      0         2                          
     upgrade_d6_block                                Idle    36     0         36                         
     upgrade_d6_file                                 Idle    683    0         683                        
     upgrade_d6_user_picture_file                    Idle    0      0         0                          
     upgrade_user_picture_field                      Idle    1      0         1                          
     upgrade_user_picture_field_instance             Idle    1      0         1                          
     upgrade_user_picture_entity_display             Idle    1      0         1                          
     upgrade_user_picture_entity_form_display        Idle    1      0         1                          
     upgrade_d6_user                                 Idle    4      0         4                          
     upgrade_d6_node_type                            Idle    6      0         6                          
     upgrade_d6_node_settings                        Idle    1      0         1                          
     upgrade_d6_field                                Idle    12     0         12                         
     upgrade_d6_field_instance                       Idle    15     0         15                         
     upgrade_d6_field_instance_widget_settings       Idle    15     0         15                         
     upgrade_d6_view_modes                           Idle    3      0         3                          
     upgrade_d6_field_formatter_settings             Idle    33     0         33                         
     upgrade_d6_upload_field                         Idle    1      0         1                          
     upgrade_d6_upload_field_instance                Idle    1      0         1                          
     upgrade_d6_node_blog                            Idle    0      0         0                          
     upgrade_d6_node_chef_profile                    Idle    3      0         3                          
     upgrade_d6_node_page                            Idle    0      0         0                          
     upgrade_d6_node_poll                            Idle    0      0         0                          
     upgrade_d6_node_recipe                          Idle    4      0         4                          
     upgrade_d6_node_story                           Idle    2      0         2                          
     upgrade_d6_comment_type                         Idle    2      0         2                          
     upgrade_d6_comment_field                        Idle    2      0         2                          
     upgrade_d6_comment_field_instance               Idle    3      0         3                          
     upgrade_d6_comment_entity_display               Idle    3      0         3                          
     upgrade_d6_comment_entity_form_display          Idle    3      0         3                          
     upgrade_d6_comment                              Idle    0      0         0                          
     upgrade_d6_comment_entity_form_display_subject  Idle    2      0         2                          
     upgrade_d6_node_revision_blog                   Idle    0      0         0                          
     upgrade_d6_node_revision_chef_profile           Idle    0      0         0                          
     upgrade_d6_node_revision_page                   Idle    0      0         0                          
     upgrade_d6_node_revision_poll                   Idle    0      0         0                          
     upgrade_d6_node_revision_recipe                 Idle    0      0         0                          
     upgrade_d6_node_revision_story                  Idle    0      0         0                          
     upgrade_d6_node_setting_promote                 Idle    6      0         6                          
     upgrade_d6_node_setting_status                  Idle    6      0         6                          
     upgrade_d6_node_setting_sticky                  Idle    6      0         6                          
     upgrade_user_profile_field                      Idle    0      0         0                          
     upgrade_user_profile_field_instance             Idle    0      0         0                          
     upgrade_user_profile_entity_display             Idle    0      0         0                          
     upgrade_user_profile_entity_form_display        Idle    0      0         0                          
     upgrade_d6_profile_values                       Idle    0      0         0                          
     upgrade_d6_taxonomy_vocabulary                  Idle    0      0         0                          
     upgrade_d6_taxonomy_term                        Idle    0      0         0                          
     upgrade_d6_upload                               Idle    0      0         0                          
     upgrade_d6_upload_entity_display                Idle    1      0         1                          
     upgrade_d6_upload_entity_form_display           Idle    1      0         1                          
     upgrade_d6_user_contact_settings                Idle    4      0         4                          
     upgrade_d6_vocabulary_field                     Idle    0      0         0                          
     upgrade_d6_vocabulary_field_instance            Idle    0      0         0                          
     upgrade_d6_vocabulary_entity_display            Idle    0      0         0                          
     upgrade_d6_vocabulary_entity_form_display       Idle    0      0         0                          
     upgrade_menu_links                              Idle    0      0         0         



<!-- .slide: data-background-color="#718522" -->
## Hybrid Workflow: Export

Export as config

    drush config-export

Everything should now be in your config folder.



<!-- .slide: data-background-color="#718522" -->
## Hybrid Workflow: Remix
Edit migrate_plus.migration.d6_user.yml

    id: d6_user
    label: User accounts
    migration_tags:
      - Drupal 6
    source:
      plugin: d6_user
    process:
      uid: uid 
      name: name
      pass: pass
      mail: mail
      created: created
      access: access
      login: login
      status: status
      timezone:
        plugin: user_update_7002
        source: timezone
      preferred_langcode: language
      init: init
      roles:
        plugin: migration
        migration: d6_user_role
        source: roles
      user_picture:
        plugin: migration
        migration: d6_user_picture_file
        source: uid 
        no_stub: true
    destination:
      plugin: entity:user
      md5_passwords: true
    migration_dependencies:
      required:
        - d6_user_role
      optional:
        - d6_user_picture_file
        - user_picture_entity_display
        - user_picture_entity_form_display



<!-- .slide: data-background-color="#718522" -->
## Hybrid Workflow: Process Chain

Implicit get plugin:

    process:
      title: subject
   
Specific process plugin: 

    process:
      uid:
        plugin: migration
        migration: users
        source: author
  
Chained process: 
d
    process:
        format:
            -
                plugin: machine_name
                source: name
            -
                plugin: dedupe_entity
                entity_type: filter_format
                field: format



<!-- .slide: data-background-color="#718522" -->
## Hybrid Workflow: run the migraitons

Import your configuration if you made changes.

    drush config-import

Run all migrations!

    drush migrate-import --all

Or run only a single migration:

    drush migrate-import d6_user



<!-- .slide: data-background-color="#718522" -->
## Hybrid Workflow: Further Reading

 - Custom Drupal-to-Drupal Migrations with Migrate Tools 
  https://drupalize.me/blog/201605/custom-drupal-drupal-migrations-migrate-tools



<!-- .slide: data-background-color="#8B7324" -->
## Manual Method

<blockquote> As a goat, I have narrowed my use case to just what I need and I would like to be all zen and do it from scratch.</blockquote>

All the details of configuring a migration.



<!-- .slide: data-background-color="#8B7324" -->
## Manual Method Setup
Migrating from everything else

 - Enable: migrate, migrate_drupal, migrate_drupal_ui, migrate_tools, migrate_plus, migrate_upgrade
 - Install drush
 - For DB migraitons, add 'migrate' database to settings.php:

     <pre><code>$databases['migrate']['default'] = array (
      'database' => 'cooks_legacy',
      'username' => 'root',
      'password' => 'my-secret-pw',
      'prefix' => '', 
      'host' => 'db',
      'port' => '3306',
      'namespace' => 'Drupal\Core\Database\Driver\mysql',
      'driver' => 'mysql',
    );</code></pre>

You can import from multiple DBs by using migration groups and corresponding
key name.



<!-- .slide: data-background-color="#8B7324" -->
## Manual Method Custom Module

We are not going to use the 'migrate' database we put into settings.php. We will
just use a text file called animals.csv.

    rcw@mojito:~/migrate_demo$ tree
    .
    ├── animals.csv
    ├── config
    │   └── install
    │       └── migrate_plus.migration.animals.yml
    └── migrate_demo.info.yml

The file only has one column.

    rcw@mojito:~/migrate_demo$ cat animals.csv 
    lion
    tiger
    elephant
    monkey




<!-- .slide: data-background-color="#8B7324" -->
## Manual method, after enabling

    # drush ms                                             
    Group: default  Status  Total  Imported  Unprocessed  Last imported 
    animals         Idle    4      0         4   

Now import!

    # drush mi animals
    Processed 4 items (4 created, 0 updated, 0 failed, 0 ignored) - done    [status]
    with 'animals'

<img src='images/csv_result.png'>



<!-- .slide: data-background-color="#8B7324" -->
## Migration represented as config
using migrate_plus

    rcw@mojito:~/migrate_demo/config/install$ cat migrate_plus.migration.animals.yml 
    # The source data is in CSV files, so we use the 'csv' source plugin.
    id: animals
    label: CSV file migration
    migration_tags:
      - CSV
    source:
      plugin: csv
      # Full path to the file.
      path: /var/www/html/modules/migration_dev/migrate_demo/animals.csv
      header_row_count: 0
      keys:
        - id
      column_names:
        0:
          id: Identifier
    destination:
      plugin: entity:node
    process:
      type:
        plugin: default_value
        default_value: article
      title: id 



<!-- .slide: data-background-color="#8B7324" -->
## Manual Method: Further Reading

 - Drupal 6 to Drupal 8(.1.x) Custom Content Migration
   https://www.drupaleasy.com/blogs/ultimike/2016/04/drupal-6-drupal-81x-custom-content-migration



<!-- .slide: data-background-color="#8B4524" -->
## Migrations are Plugins

<blockquote> As a goat, I want to know every detail of what I am working with, extend it in new ways, and contribute back to the community.</blockquote>

A brief introduction to plugins.



<!-- .slide: data-background-color="#8B4524" -->
## This whole talk was about Plugins

Who knew?

    rcw@mojito:~/katli/repos/drupal/core/modules/migrate_drupal$ tree
    .
    ├── migrate_drupal.info.yml
    ├── migrate_drupal.module
    ├── migrate_drupal.services.yml
    ├── src
    │   ├── Annotation
    │   │   └── MigrateCckField.php
    │   ├── MigrationConfigurationTrait.php
    │   ├── MigrationCreationTrait.php
    │   ├── Plugin
    │   │   ├── migrate
    │   │   │   ├── cckfield
    │   │   │   │   └── CckFieldPluginBase.php
    │   │   │   ├── CckMigration.php
    │   │   │   ├── destination
    │   │   │   │   └── EntityFieldStorageConfig.php
    │   │   │   └── source
    │   │   │       ├── d6
    │   │   │       │   └── i18nVariable.php
    │   │   │       ├── d7
    │   │   │       │   └── FieldableEntity.php
    │   │   │       ├── DrupalSqlBase.php
    │   │   │       ├── EmptySource.php
    │   │   │       ├── VariableMultiRow.php
    │   │   │       └── Variable.php
    │   │   └── MigrateCckFieldInterface.php
    │   └── Tests
    │       └── StubTestTrait.php
    ...



<!-- .slide: data-background-color="#8B4524" -->
## Plugin Structure

Single file, no registration, just rebuild cache.

    rcw@mojito:~/katli/repos/drupal/core/modules/migrate/src/Plugin/migrate/process$ cat Concat.php 
    <?php
    
    namespace Drupal\migrate\Plugin\migrate\process;
    
    use Drupal\migrate\MigrateException;
    use Drupal\migrate\MigrateExecutableInterface;
    use Drupal\migrate\ProcessPluginBase;
    use Drupal\migrate\Row;
    
    /**
     * Concatenates the strings in the current value.
     *
     * @MigrateProcessPlugin(
     *   id = "concat",
     *   handle_multiples = TRUE
     * )
     */
    class Concat extends ProcessPluginBase {
    
      /**
       * {@inheritdoc}
       *
       * Concatenates the strings in the current value.
       */
      public function transform($value, MigrateExecutableInterface $migrate_executable, Row $row, $destination_property) {
        if (is_array($value)) {
          $delimiter = isset($this->configuration['delimiter']) ? $this->configuration['delimiter'] : '';
          return implode($delimiter, $value);
        }
        else {
          throw new MigrateException(sprintf('%s is not an array', var_export($value, TRUE)));
        }
      }
    
    }

Remember to update the annotation (comment) as it is processed.



<!-- .slide: data-background-color="#8B4524" -->
## Plugins: Further Reading

 - Migrate API in Drupal 8
   https://www.drupal.org/node/2127611




<!-- .slide: data-background-color="#EBEBEB" -->
## Questions?

https://twitter.com/ryan_weal

https://twitter.com/italiatina

<img src=images/kafei.png>
The Kafei Interactive logo and wordmark is a trademark of Kafei Interactive Inc. (Canada).
