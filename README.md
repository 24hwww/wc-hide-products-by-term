# Hide products by term or attribute in WooCommerce
Hide products by term or attribute in woocommerce
```php
if (!class_exists('hidden_products_taxonomy_by_term')) {
class hidden_products_taxonomy_by_term{
    private static $_instance = null;
    public static function instance() {
        return is_null( self::$_instance ) ? self::$_instance = new self() : self::$_instance;
    }
	public function __construct() {
	}
	
	public function get_all_taxonomies_woocommerce(){
		$output = [];
		if ( in_array( 'woocommerce/woocommerce.php', apply_filters( 'active_plugins', get_option( 'active_plugins' ) ) ) ) {
			$output = array_keys(get_taxonomies(['object_type' => array( 'product' )], 'names' ));
		}
		return $output;
	}
	
	public static function init() {
		$instance = self::instance();
		$taxonomies = $instance->get_all_taxonomies_woocommerce();
		
		if(is_array($taxonomies) && count($taxonomies) > 0){
			foreach($taxonomies as $taxonomy){
				add_action( "{$taxonomy}_edit_form_fields", [$instance,'add_field_hidden_or_not_func'], 10, 2 );
				add_action( "{$taxonomy}_add_form_fields", [$instance,'add_field_hidden_or_not_func'], 10, 2 );
			}
				add_action( 'woocommerce_after_add_attribute_fields', [$instance,'add_field_hidden_or_not_func'], 10, 2 );
				add_action( 'woocommerce_after_edit_attribute_fields', [$instance,'add_field_hidden_or_not_func'], 10, 2 );
				add_action( 'woocommerce_attribute_added', [$instance,'save_custom_field_wc_attribute_func'], 10, 2 );
				add_action( 'woocommerce_attribute_updated', [$instance,'save_custom_field_wc_attribute_func'], 10, 2 );			
		}
		

		add_shortcode( 'debug_plugin_dev', [$instance,'debug_plugin_dev_func'] );

	}
	
	public function add_field_hidden_or_not_func($term='', $taxonomy=''){
	$term_id = !empty($term->term_id) ? $term->term_id : '';
	$check_field = get_term_meta($term_id, 'hidden_products_taxonomy_by_term', true ); 
	?>
	<tr class="form-field">
		<th><label for="hidden_products_taxonomy_by_term">Ocultar productos que tengan este termino o atributo.</label></th>
		<td>
			<div class="form-field term-description-wrap">
			<label>
				<input id="hidden_products_taxonomy_by_term" type="checkbox" name="hidden_products_taxonomy_by_term" /> Ocultar productos que tengan este termino o atributo.
			</label>
				</div>
		</td>
	</tr>
	<?php
	}
	
	public function save_custom_field_wc_attribute_func($term_id){
		if ( !is_admin() ){ return false; } 
		$hidden_product = isset($_POST['hidden_products_taxonomy_by_term']) ? esc_attr($_POST['hidden_products_taxonomy_by_term']) : '';
		update_term_meta($term_id, 'wc_hidden_product_by_term', $hidden_product);
	}
	
	public function debug_plugin_dev_func(){
		print_r($this->get_all_taxonomies_woocommerce());
	}
	
}
add_action( 'admin_init', ['hidden_products_taxonomy_by_term', 'init']);
}
```
