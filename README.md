# collection-swatch-add-to-cart
<!----custom-color-swatches-start-------->
  <div class="aa custom-swatches-list" id="color-swatch-{{ card_product.id }}">
  {% for option in card_product.options_with_values %}
    {% if option.name contains 'Color' %}
      {% for value in option.values %}
        {% assign variant = card_product.variants | where: 'option1', value | first %}
        
        {% if variant and variant.image %}
          <label 
            class="color-change" 
            data-image="{{ variant.image | img_url: 'original' }}"
            data-id="{{ card_product.id }}"
            data-variant-id="{{ variant.id }}"
            style="background-color: {{ value | handleize }}; color: transparent; cursor: pointer;">
            {{ value }}
          </label>
        {% endif %}
      {% endfor %}
    {% endif %}
  {% endfor %}
</div>
<!----custom-color-swatches end t-------->
<!----custom-add-cart-------->
<div class="custom-cart-button banner__buttons">
  <button class="custom-add-to-cart button button--primary" data-product-id="{{ card_product.id }}">Add to Cart</button>
  <input type="hidden" id="selected-variant-id-{{ card_product.id }}" name="selected_variant_id" value="{{ card_product.first_available_variant.id }}">
  
  <!-- Cart Message Area -->
  <div class="cart-message" id="cart-message-{{ card_product.id }}"></div>
</div>

<script>
  $(document).ready(function(){
  // Listen for clicks on any color swatch
  $('.color-change').on('click', function() {
    // Get the clicked swatch's data attributes
    var productId = $(this).data('id');
    var variantId = $(this).data('variant-id');
    var variantImage = $(this).data('image');
    
    // Update the hidden input with the selected variant ID
    $('#selected-variant-id-' + productId).val(variantId);
    
    // Update the feature image
    $('#' + productId).attr('srcset', variantImage);
    $('#' + productId).attr('src', variantImage);
    
    // Optional: Show a message or visual effect to indicate selection
    $(this).addClass('selected').siblings().removeClass('selected');
  });

  // Listen for clicks on Add to Cart button
  $('.custom-add-to-cart').on('click', function(e) {
    e.preventDefault();
    
    // Get the product ID
    var productId = $(this).data('product-id');
    // Get the selected variant ID from the hidden input field
    var variantId = $('#selected-variant-id-' + productId).val();
    var quantity = 1;

    // Add to cart AJAX request
    $.ajax({
      type: 'POST',
      url: '/cart/add.js',
      data: {
        quantity: quantity,
        id: variantId
      },
      dataType: 'json',
      success: function(response) {
        // Show success message
        $('#cart-message-' + productId).html('<p>Product added to cart!</p>');
        
        // Update the cart icon bubble with the latest item count
        $.ajax({
          type: 'GET',
          url: '/cart.js',
          dataType: 'json',
          success: function(cart) {
            // Update the cart icon and item count
            $('#cart-icon-bubble').html(`
              <div class="icon-cart">
                <span class="svg-wrapper">
                  <svg xmlns="http://www.w3.org/2000/svg" fill="none" class="icon icon-cart" viewBox="0 0 40 40">
                    <path fill="currentColor" fill-rule="evenodd" d="M20.5 6.5a4.75 4.75 0 0 0-4.75 4.75v.56h-3.16l-.77 11.6a5 5 0 0 0 4.99 5.34h7.38a5 5 0 0 0 4.99-5.33l-.77-11.6h-3.16v-.57A4.75 4.75 0 0 0 20.5 6.5m3.75 5.31v-.56a3.75 3.75 0 1 0-7.5 0v.56zm-7.5 1h7.5v.56a3.75 3.75 0 1 1-7.5 0zm-1 0v.56a4.75 4.75 0 1 0 9.5 0v-.56h2.22l.71 10.67a4 4 0 0 1-3.99 4.27h-7.38a4 4 0 0 1-4-4.27l.72-10.67z"></path>
                  </svg>
                </span>
                <div class="cart-count-bubble">${cart.item_count}</div>
              </div>
            `);
          }
        });
        
        // Hide the success message after a short delay
        setTimeout(function() {     
          $('#cart-message-' + productId).fadeOut(); 
        }, 1000);
      },
      error: function() {
        $('#cart-message-' + productId).html('<p>Sorry, there was an error. Please try again.</p>');
      }
    });
  });
});
</script>
