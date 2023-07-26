<?php
/**
 * Plugin Name:         WordLift Add-On for WP All Import
 * Plugin URI:          https://wordlift.io
 * Description:         Import data into WordLift with WP All Import.
 * Version:             1.0.1
 * Requires at least:   4.3
 * Requires PHP:        5.3
 * Author:              WordLift
 * Text Domain:         wordlift-add-on-for-wp-all-import
 * Domain Path:         /languages
 * License:             GPLv2 or later
 * License URI:         https://www.gnu.org/licenses/gpl-2.0.html
 */


// We start after the plugins have been loaded to ensure that WordLift is installed.
use Wordlift\Content\Wordpress\Wordpress_Content_Id;
use Wordlift\Content\Wordpress\Wordpress_Content_Service;

add_action( 'plugins_loaded', '__wpai_wl_addon__plugins_loaded' );

/**
 * Bootstrap the plugin, we're hooked to `plugins_loaded` because we depend on WordLift to be installed and activated.
 *
 * @return void
 */
function __wpai_wl_addon__plugins_loaded() {

	// Bail out if we're not in admin, cli or importing.
	if ( ! is_admin() && php_sapi_name() !== 'cli' && ! isset( $_GET['import_key'] ) ) {
		return;
	}

	// Bail out if the requirements arent' satisfied.
	if ( ! __wpai_wl_addon__requirements() ) {
		__wpai_wl_addon__notice();

		return;
	}

	include "rapid-addon.php";

	$addon = new RapidAddon( 'WordLift', 'wordlift_addon' );
	$addon->add_field(
		'entity__rel_uri',
		__( 'Relative Item Id', 'wordlift-add-on-for-wp-all-import' ),
		'text',
		null,
		__( 'The Item Id relative to the dataset URI, e.g. articles/headline_1.', 'wordlift-add-on-for-wp-all-import' ),
		false
	);
	$addon->set_import_function( '__wpai_wl_addon__import' );

	$addon->run( array(
		// Only run this add-on when importing to a supported entity post type.
		'post_type' => Wordlift_Entity_Service::valid_entity_post_types()
	) );
}

/**
 * Check if WordLift and the required classes are installed and available.
 * @return bool
 */
function __wpai_wl_addon__requirements() {

	return class_exists( 'Wordlift_Entity_Service' )
	       && class_exists( 'Wordlift\Content\Wordpress\Wordpress_Content_Service' )
	       && class_exists( 'Wordlift\Content\Wordpress\Wordpress_Content_Id' );
}

/**
 * Display a notice to the user about missing requirements.
 * @return void
 */
function __wpai_wl_addon__notice() {
	add_action( 'admin_notices', function () {
		?>
        <div class="notice notice-error is-dismissible">
            <p><?php _e( 'WP All Import - WordLift Add-On requires WordLift to be installed and activate, please check the plugins page.', 'wordlift-add-on-for-wp-all-import' ); ?></p>
        </div>
		<?php
	} );
}

/**
 * Import data into a post, called by WP All Import.
 * @return void
 */
function __wpai_wl_addon__import( $post_id, $data, $options ) {

	try {
        // Replace spaces with underscores. Spaces will cause the entity not to sync to the remote KG.
		$rel_uri         = str_replace( ' ', '_', $data['entity__rel_uri'] );
		$content_service = Wordpress_Content_Service::get_instance();
		$content_id      = Wordpress_Content_Id::create_post( $post_id );
		$content_service->set_entity_id( $content_id, $rel_uri );
	} catch ( Exception $e ) {
		__wpai_wl_addon__logger( "An error occurred while importing {$rel_uri} to $post_id: {$e->getMessage()}" );
	}

}

/**
 * Log a message in WPAI logs.
 *
 * @param string $message
 *
 * @return void
 */
function __wpai_wl_addon__logger( $message ) {
	printf( "[%s] $message", date( "H:i:s" ) );
	flush();
}
