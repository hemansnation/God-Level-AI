(function($){
	$(document).ready(function() {
		var $locked_containers = [];

		$( '.et_bloom_make_form_visible' ).removeAttr( 'style' );

		$( '.et_bloom_custom_html_form input[type="radio"], .et_bloom_custom_html_form input[type="checkbox"]' ).uniform();

		$( 'body' ).on( 'click', 'span.et_bloom_close_button', function(){
			perform_popup_closing( $( this ).closest( '.et_bloom_optin' ) );

			return false;
		});

		function toggle_checkbox(element, value) {
			$(element).children('i').toggleClass('et_bloom_icon et_bloom_icon_check');
		}

		$('body .et_bloom_custom_field_checkbox input[type=checkbox]:checked').each(function() {
			toggle_checkbox($(this).next('label'));
		});

		$('body').on('click', '.et_bloom_custom_field_checkbox label', function() {
			toggle_checkbox(this);
		});

		function perform_popup_closing( $popup_container ) {
			$popup_container.addClass( 'et_bloom_exit_animation' );

			setTimeout( function() {
				if ( $popup_container.hasClass( 'et_bloom_trigger_click' ) ) {
					$popup_container.removeClass( 'et_bloom_visible et_bloom_animated et_bloom_exit_animation' );
				} else {
					$popup_container.remove();
				}
			}, 400 );

			$( 'body' ).removeClass( 'et_bloom_popup_active' );

		}

		function update_stats_table( type, $this_button ) {
			// do not update stats if visitor logged in
			if ( 'logged' === bloomSettings.is_user_logged_in ) {
				return;
			}

			var $optin_id = typeof $this_button.data( 'current_optin_id' ) !== 'undefined' ? $this_button.data( 'current_optin_id' ) : $this_button.data( 'optin_id' ),
				$page_id = $this_button.data( 'page_id' ),
				$list_id = $this_button.data( 'list_id' );

			var cookie = 'et_bloom_optin_'.concat( $optin_id, '_', $list_id, '_', type );

			if ( cookieExists( cookie ) ) {
				return;
			}

			$stats_data = JSON.stringify({ 'type' : type, 'optin_id' : $optin_id, 'page_id' : $page_id, 'list_id' : $list_id });

			$.ajax({
				type: 'POST',
				url: bloomSettings.ajaxurl,
				data: {
					action : 'bloom_handle_stats_adding',
					stats_data_array : $stats_data,
					update_stats_nonce : bloomSettings.stats_nonce
				}
			}).done( function() {
				set_cookie( 365, cookie.concat( '=true' ) );
			});
		}

		function setCookieExpire( days ) {
			var ms = days*24*60*60*1000;

			var date = new Date();
			date.setTime( date.getTime() + ms );

			return "; expires=" + date.toUTCString();
		}

		function checkCookieValue( cookieName, value ) {
			return parseCookies()[cookieName] == value;
		}

		function cookieExists( cookie_name ) {
			return 'undefined' !== typeof parseCookies()[cookie_name];
		}

		function parseCookies() {
			var cookies = document.cookie.split( '; ' );

			var ret = {};
			for ( var i = cookies.length - 1; i >= 0; i-- ) {
			  var el = cookies[i].split( '=' );
			  ret[el[0]] = el[1];
			}
			return ret;
		}

		function set_cookie( $expire, $cookie_content ) {
			var $cookie_content = '' == $cookie_content ? 'etBloomCookie=true' : $cookie_content;
			cookieExpire = setCookieExpire( $expire );
			document.cookie = $cookie_content + cookieExpire + "; path=/";
		}

		function get_url_parameter( param_name ) {
			var page_url = window.location.search.substring(1);
			var url_variables = page_url.split('&');
			for ( var i = 0; i < url_variables.length; i++ ) {
					var curr_param_name = url_variables[i].split( '=' );
				if ( curr_param_name[0] == param_name ) {
					return curr_param_name[1];
				}
			}
		}

		//separate function for the setTimeout to make it work properly within the loop.
		function make_popup_visible( $popup, $delay, $cookie_exp, $cookie_content ){
			if ( ! $popup.hasClass( 'et_bloom_visible' ) ) {
				setTimeout( function() {
					$popup.addClass( 'et_bloom_visible et_bloom_animated' );
					$stats_data_container = 0 != $popup.find( '.et_bloom_custom_html_form' ).length ? $popup.find( '.et_bloom_custom_html_form' ) : $popup.find( '.et_bloom_submit_subscription' );
					update_stats_table( 'imp', $stats_data_container );

					if ( '' != $cookie_exp ) {
						set_cookie( $cookie_exp, $cookie_content );
					}

					if ( $( '.et_bloom_resize' ).length ) {
						$( '.et_bloom_resize.et_bloom_visible' ).each( function() {
							define_popup_position( $( this ), true, 0 );
						});
					}

					display_image( $popup );

				}, $delay );
			}
		}

		function display_image( $popup ) {
			setTimeout( function() {
				$popup.find( '.et_bloom_image' ).addClass( 'et_bloom_visible_image' );
			}, 500 );
		}

		function auto_popup( $current_popup_auto, $delay ) {
			var $data_holder = $current_popup_auto.find( '.et_bloom_custom_html_form' ).length ? $current_popup_auto.find( '.et_bloom_custom_html_form' ) : $current_popup_auto.find( '.et_bloom_submit_subscription' ),
				page_id = $data_holder.data( 'page_id' ),
				optin_id = $data_holder.data( 'optin_id' ),
				list_id = $data_holder.data( 'list_id' );

			if ( ! $current_popup_auto.hasClass( 'et_bloom_animated' ) ) {
				var $cookies_expire_auto = $current_popup_auto.data( 'cookie_duration' ) ? $current_popup_auto.data( 'cookie_duration' ) : false,
					$already_subscribed = checkCookieValue( 'et_bloom_subscribed_to_' + optin_id + list_id, 'true' );

				if ( ( ( false !== $cookies_expire_auto && ! checkCookieValue( 'etBloomCookie_' + optin_id, 'true' ) ) || false == $cookies_expire_auto ) && ! $already_subscribed ) {
					if ( false !== $cookies_expire_auto ) {
						make_popup_visible ( $current_popup_auto, $delay, $cookies_expire_auto, 'etBloomCookie_' + optin_id + '=true' );
					} else {
						make_popup_visible ( $current_popup_auto, $delay, '', '' );
					}
				}
			}
		}

		function scroll_trigger( $current_popup_bottom, is_bottom_trigger ) {
			var triggered = 0,
				$data_holder = $current_popup_bottom.find( '.et_bloom_custom_html_form' ).length ? $current_popup_bottom.find( '.et_bloom_custom_html_form' ) : $current_popup_bottom.find( '.et_bloom_submit_subscription' ),
				page_id = $data_holder.data( 'page_id' ),
				optin_id = $data_holder.data( 'optin_id' );
				list_id = $data_holder.data( 'list_id' );

			if ( ! $current_popup_bottom.hasClass( 'et_bloom_animated' ) ) {
				var	cookies_expire_bottom = $current_popup_bottom.data( 'cookie_duration' ) ? $current_popup_bottom.data( 'cookie_duration' ) : false,
					$already_subscribed = checkCookieValue( 'et_bloom_subscribed_to_' + optin_id + list_id, 'true' );

				if ( true == is_bottom_trigger ) {
					var scroll_trigger = $( '.et_bloom_bottom_trigger' ).length ? $( '.et_bloom_bottom_trigger' ).offset().top : $( document ).height() - 500;
				} else {
					var scroll_pos = $current_popup_bottom.data( 'scroll_pos' ) > 100 ? 100 : $current_popup_bottom.data( 'scroll_pos' ),
						scroll_trigger = 100 == scroll_pos ? $( document ).height() - 50 : $( document ).height() * scroll_pos / 100;
				}

				$( window ).scroll( function(){
					if ( ( ( false !== cookies_expire_bottom && ! checkCookieValue( 'etBloomCookie_' + optin_id, 'true' ) ) || false == cookies_expire_bottom ) && ! $already_subscribed ) {
						if( $( window ).scrollTop() + $( window ).height() > scroll_trigger ) {
							if ( 0 == triggered ) {
								if ( false !== cookies_expire_bottom ) {
									make_popup_visible ( $current_popup_bottom, 0, cookies_expire_bottom, 'etBloomCookie_' + optin_id + '=true' );
								} else {
									make_popup_visible ( $current_popup_bottom, 0, '', '' );
								}

								triggered++;
							}
						}
					}
				});
			}
		}

		$.fn.isInViewport = function() {
			var elementTop     = $( this ).offset().top;
			var elementBottom  = elementTop + $( this ).outerHeight();
			var viewportTop    = $( window ).scrollTop();
			var viewportBottom = viewportTop + $( window ).height();

			return elementBottom > viewportTop && elementTop < viewportBottom;
		};

		var $inline_optins = $( '.et_bloom_inline_form .et_bloom_submit_subscription, .et_bloom_widget_content .et_bloom_submit_subscription, .et_bloom_custom_html_form' );

		if ( $inline_optins.length > 0 ) {
			var imp_recorded_count = 0;

			$(window).on( 'scroll.et_bloom_impressions', function() {
				$inline_optins.each( function() {
					if ( ! $(this).hasClass( 'et_bloom_impression_recorded' ) && $(this).isInViewport() ) {
						$( this ).addClass( 'et_bloom_impression_recorded' );

						imp_recorded_count++;

						update_stats_table( 'imp', $(this) );
					}
				} );

				if ( imp_recorded_count >= $inline_optins.length ) {
					$(window).off( 'scroll.et_bloom_impressions' );
				}
			} );
		}

		 if( $( '.et_bloom_auto_popup' ).length ) {
			$( '.et_bloom_auto_popup:not(.et_bloom_visible)' ).each( function() {
				var this_el = $( this ),
					delay = '' !== this_el.data( 'delay' ) ? this_el.data( 'delay' ) * 1000 : 0;
				auto_popup( this_el, delay );
			});
		 }

		if( $( '.et_bloom_trigger_bottom' ).length ) {

			$( '.et_bloom_trigger_bottom:not(.et_bloom_visible)' ).each( function(){
				scroll_trigger( $( this ), true );
			});

		}

		if( $( '.et_bloom_scroll' ).length ) {

			$( '.et_bloom_scroll:not(.et_bloom_visible)' ).each( function(){
				scroll_trigger( $( this ), false );
			});
		}

		if ( $( '.et_bloom_trigger_click' ).length ) {
			$( '.et_bloom_trigger_click:not(.et_bloom_visible)' ).each( function() {
				var $this_el = $( this ),
					selector = $this_el.attr( 'data-trigger_click' );

				if ( typeof selector !== 'undefined' ) {
					$( 'body' ).on( 'click', selector, function() {
						make_popup_visible ( $this_el, 0, '', '' );
						return false;
					});
				}
			});
		}

		if( $( '.et_bloom_trigger_idle' ).length ) {
			$( '.et_bloom_trigger_idle:not(.et_bloom_visible)' ).each( function() {
				var $this_el = $( this ),
					$data_holder = $this_el.find( '.et_bloom_custom_html_form' ).length ? $this_el.find( '.et_bloom_custom_html_form' ) : $this_el.find( '.et_bloom_submit_subscription' ),
					page_id = $data_holder.data( 'page_id' ),
					optin_id = $data_holder.data( 'optin_id' ),
					list_id = $data_holder.data( 'list_id' );

				if ( ! $this_el.hasClass( 'et_bloom_animated' ) ) {
					var $cookies_expire_idle = $this_el.data( 'cookie_duration' ) ? $this_el.data( 'cookie_duration' ) : false,
						$already_subscribed = checkCookieValue( 'et_bloom_subscribed_to_' + optin_id + list_id, 'true' );
						$idle_timeout = '' !== $this_el.data( 'idle_timeout' ) ? $this_el.data( 'idle_timeout' ) * 1000 : 30000,
						$delay = 0;

					if ( ( ( false !== $cookies_expire_idle && ! checkCookieValue( 'etBloomCookie_' + optin_id, 'true' ) ) || false == $cookies_expire_idle ) && ! $already_subscribed ) {
						$( document ).idleTimer( $idle_timeout );

						$( document ).on( 'idle.idleTimer', function() {
							if ( false !== $cookies_expire_idle ) {
								make_popup_visible ( $this_el, $delay, $cookies_expire_idle, 'etBloomCookie_' + optin_id + '=true' );
							} else {
								make_popup_visible ( $this_el, $delay, '', '' );
							}
						});
					}
				}
			});
		}

		if ( 'true' == get_url_parameter( 'et_bloom_popup' ) ) {
			$( '.et_bloom_after_comment' ).each( function() {
				auto_popup( $( this ), 0 );
			});
		}

		if ( $( '.et_bloom_after_order' ).length ) {
			$( '.et_bloom_after_purchase' ).each( function() {
				auto_popup( $( this ), 0 );
			});
		}

		if( $( '.et_bloom_locked_container' ).length ) {
			var $i = 0;

			$( '.et_bloom_locked_container' ).each( function() {
				var $this_el = $( this ),
					content = $this_el.find( '.et_bloom_locked_content' ),
					form = $this_el.find( '.et_bloom_locked_form' ),
					page_id = $this_el.data( 'page_id' ),
					optin_id = $this_el.data( 'optin_id' );

				$this_el.data( 'container_id', $i );
				$locked_containers.push( content );

				if ( checkCookieValue( 'et_bloom_unlocked' + optin_id + page_id, 'true' ) ) {
					content.css( {'display' : 'block'} );
					form.remove();
				} else {
					content.remove();
					update_stats_table( 'imp', $this_el );
				}

				$i++;
			});
		}

		$( 'body' ).on( 'click', '.et_bloom_locked_container .et_bloom_submit_subscription', function(){
			var $current_container = $( this ).closest( '.et_bloom_locked_container' ),
				container_id = $current_container.data( 'container_id' ),
				page_id = $current_container.data( 'page_id' ),
				optin_id = typeof $current_container.data( 'current_optin_id' ) !== 'undefined' ? $current_container.data( 'current_optin_id' ) : $current_container.data( 'optin_id' );

			perform_subscription( $( this ), $current_container, container_id, page_id, optin_id );

			return false;
		});

		// unlock content immediately if custom HTML form is used.
		$( 'body' ).on( 'click', '.et_bloom_locked_container .et_bloom_custom_html_form input[type="submit"], .et_bloom_locked_container .et_bloom_custom_html_form button[type="submit"]', function() {
			var current_container = $( this ).closest( '.et_bloom_locked_container' ),
				container_id = current_container.data( 'container_id' ),
				page_id = current_container.data( 'page_id' ),
				optin_id = current_container.data( 'optin_id' );

			unlock_content( current_container, container_id, page_id, optin_id );
		} );

		function unlock_content( current_container, container_id, locked_page_id, locked_optin_id ) {
			set_cookie( 365, 'et_bloom_unlocked' + locked_optin_id + locked_page_id + '=true' );
			current_container.find( '.et_bloom_locked_form' ).replaceWith( $locked_containers[container_id] );
			current_container.find( '.et_bloom_locked_content' ).css( { 'display' : 'block' } );
		}

		// Move inline forms into appropriate sections in Divi theme
		if( $( '.et_bloom_below_post' ).length ) {
			if ( $( 'body' ).hasClass( 'et_pb_pagebuilder_layout' ) ) {
				var bottom_inline = $( '.et_bloom_below_post' ),
					divi_container = '<div class="et_pb_row"><div class="et_pb_column et_pb_column_4_4"></div></div>';

				if ( bottom_inline.length ) {
					$( '.et_pb_section' ).not( '.et_pb_fullwidth_section' ).last().append( divi_container ).find( '.et_pb_row' ).last().find( '.et_pb_column' ).append( bottom_inline );
				}
			}
		}

		function define_popup_position( $this_popup, $just_loaded, $message_space ) {
			var this_popup = $this_popup.find( '.et_bloom_form_container' ),
				popup_max_height = this_popup.hasClass( 'et_bloom_popup_container' ) ? $( window ).height() - 40 : $( window ).height() - 20,
				real_popup_height = 0,
				percentage = this_popup.parent().hasClass( 'et_bloom_flyin' ) ? 0.03 : 0.05,
				percentage = this_popup.hasClass( 'et_bloom_with_border' ) ? percentage + 0.03 : percentage,
				breakout_offset = this_popup.hasClass( 'breakout_edge' ) ? 0.95 : 1,
				dashed_offset = this_popup.hasClass( 'et_bloom_border_dashed' ) ? 4 : 0,
				form_height = this_popup.find( 'form' ).innerHeight() + $message_space,
				form_add = true == $just_loaded ? 5 : 0;

			this_popup.css( { 'max-height' : popup_max_height } );

			if ( this_popup.hasClass( 'et_bloom_popup_container' ) ) {
				var top_position = $( window ).height() / 2 - this_popup.innerHeight() / 2;
				this_popup.css( { 'top' : top_position + 'px' } );
			}

			this_popup.find( '.et_bloom_form_container_wrapper' ).css( { 'max-height' : popup_max_height - 20 } );


			if ( ( 768 > $( 'body' ).outerWidth() + 15 ) || this_popup.hasClass( 'et_bloom_form_bottom' ) ) {
				if ( this_popup.hasClass( 'et_bloom_form_right' ) || this_popup.hasClass( 'et_bloom_form_left' ) ) {
					this_popup.find( '.et_bloom_form_header' ).css( { 'height' : 'auto' } );
				}

				real_popup_height = this_popup.find( '.et_bloom_form_container_wrapper' ).height() + 30 + form_add;

				if ( this_popup.hasClass( 'et_bloom_form_right' ) || this_popup.hasClass( 'et_bloom_form_left' ) ) {
					this_popup.find( '.et_bloom_form_container_wrapper' ).css( { 'height' : real_popup_height - 30 + dashed_offset } );
				}
			} else {
				real_popup_height = this_popup.find( '.et_bloom_form_container_wrapper' ).height() + $message_space;

				/**
				 * Sometimes real_popup_height return 0 because the page is no loaded fully. This
				 * is added to set the Bloom container height as 100% to avoid 0 height. Note, the
				 * height will be set as 100% only when the real_popup_height is 0.
				 */
				var container_height = real_popup_height;
				if (parseInt(real_popup_height) === 0) {
					container_height = '100%';
				}

				if ( this_popup.hasClass( 'et_bloom_form_right' ) || this_popup.hasClass( 'et_bloom_form_left' ) ) {
					this_popup.find( '.et_bloom_form_header' ).css( { 'height' : real_popup_height * breakout_offset - dashed_offset } );
					this_popup.find( '.et_bloom_form_content' ).css( { 'min-height' : real_popup_height - dashed_offset } );
					this_popup.find( '.et_bloom_form_container_wrapper' ).css( { 'height' : container_height } );
				}
			}

			if ( real_popup_height > popup_max_height ) {
				this_popup.find( '.et_bloom_form_container_wrapper' ).addClass( 'et_bloom_vertical_scroll' );
			} else {
				this_popup.find( '.et_bloom_form_container_wrapper' ).removeClass( 'et_bloom_vertical_scroll' );
			}

			if ( $this_popup.hasClass( 'et_bloom_popup' ) ) {
				$( 'body' ).addClass( 'et_bloom_popup_active' );
			}
		}

		$( 'body' ).on( 'click', '.et_bloom_submit_subscription:not(.et_bloom_submit_subscription_locked)', function() {
			perform_subscription( $( this ), '', '', '', '' );
			return false;
		});

		function perform_subscription(this_button, current_container, container_id, locked_page_id, locked_optin_id) {
			var this_form = this_button.parents('form');
			var list_id = this_button.data('list_id');
			var account_name = this_button.data('account');
			var service = this_button.data('service');
			var name = this_form.find('.et_bloom_subscribe_name input').val();
			var last_name = undefined !== this_form.find('.et_bloom_subscribe_last input').val() ? this_form.find('.et_bloom_subscribe_last input').val() : '';
			var email = this_form.find('.et_bloom_subscribe_email input').val();
			var page_id = this_button.data('page_id');
			var optin_id = this_button.data('optin_id');
			var $popup_container = this_form.closest('.et_bloom_optin');
			var is_popup = $popup_container.hasClass('et_bloom_popup') || $popup_container.hasClass('et_bloom_flyin');
			var $success_action_el = this_button.closest('.et_bloom_success_action');
			var success_action_details = $success_action_el.length > 0 ? $success_action_el.parent().data('success_action_details').split('|') : [];
			var success_action = 2 === success_action_details.length ? success_action_details[0] : '';
			var success_action_url = '' !== success_action ? success_action_details[1] : '';

			var ip_address        = this_button.data('ip_address');
			var $fields_container = this_form.find('.et_bloom_fields');
			var $custom_fields    = $fields_container.find('.et_bloom_custom_field input[type=text], .et_bloom_checkbox_handle, [data-field_type="radio"], textarea, select');
			var custom_fields     = {};
			var is_valid          = true;

			// Email Validation
			// Use the regex defined in the HTML5 spec for input[type=email] validation
			// (see https://www.w3.org/TR/2016/REC-html51-20161101/sec-forms.html#email-state-typeemail)
			var et_email_reg_html5 = /^[a-zA-Z0-9.!#$%&'*+\/=?^_`{|}~-]+@[a-zA-Z0-9](?:[a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?(?:\.[a-zA-Z0-9](?:[a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?)*$/;

			this_form.find('.et_bloom_warn_field').removeClass('et_bloom_warn_field');

			if (this_form.length > 0 && typeof this_form[0].reportValidity === 'function') {
				// Checks HTML5 validation constraints
				is_valid = this_form[0].reportValidity();
			}

			$custom_fields.each( function() {
				var $this_el      = $(this);
				var $this_wrapper = false;
				var this_id       = $this_el.attr('data-id');
				var field_type    = typeof $this_el.data( 'field_type' ) !== 'undefined' ? $this_el.data( 'field_type' ) : 'text';

				if ('checkbox' === field_type || 'radio' === field_type) {
					$this_wrapper = $this_el.closest('.et_bloom_custom_field');
				}

				var this_val      = $this_el.val();
				var required_mark = typeof $this_el.data('required_mark') !== 'undefined' ? $this_el.data('required_mark') : 'not_required';
				var original_id   = typeof $this_el.data('original_id') !== 'undefined' ? $this_el.data('original_id') : '';
				var unchecked     = false;
				var default_value;

				// radio field properties adjustment
				if ('radio' === field_type) {
					if (0 !== $this_wrapper.find('input[type="radio"]').length) {
						var $firstRadio = $this_wrapper.find('input[type="radio"]:first');

						required_mark = typeof $firstRadio.data('required_mark') !== 'undefined' ? $firstRadio.data('required_mark') : 'not_required';

						this_val = '';

						if ($this_wrapper.find('input[type="radio"]:checked')) {
							this_val = $this_wrapper.find('input[type="radio"]:checked').val();
						}
					}

					if (!$.isEmptyObject(this_val)) {
						custom_fields[this_id] = this_val;
					}

					if (0 === $this_wrapper.find('input[type="radio"]:checked').length) {
						unchecked = true;
					}

					if (this_val) {
						custom_fields[this_id] = this_val;
					}
				} else if ('checkbox' === field_type) {
					this_val = {};

					if ( 0 !== $this_wrapper.find('input[type="checkbox"]').length) {
						var $checkboxHandle = $this_wrapper.find('.et_bloom_checkbox_handle');

						required_mark = typeof $checkboxHandle.data('required_mark') !== 'undefined' ? $checkboxHandle.data('required_mark') : 'not_required';

						if ($this_wrapper.find('input[type="checkbox"]:checked').length > 0) {
							$this_wrapper.find('input[type="checkbox"]:checked').each(function() {
								var field_id       = $(this).data('id');
								this_val[field_id] = $(this).val();
							} );
						}
					}

					if (!$.isEmptyObject(this_val)) {
						custom_fields[this_id] = this_val;
					}

					if (0 === $this_wrapper.find('input[type="checkbox"]:checked').length) {
						unchecked = true;
					}
				} else if ('ontraport' === service && 'select' === field_type) {
					// Need to pass option ID as a value for dropdown menu in Ontraport
					var $selected_option   = $this_el.find(':selected');
					custom_fields[this_id] = $selected_option.length > 0 ? $selected_option.data('id') : this_val;
				} else {
					custom_fields[this_id] = this_val;
				}

				// add error message for the field if it is required and empty
				if ('required' === required_mark && ('' === this_val || true === unchecked)) {
					if (false === $this_wrapper) {
						$this_el.addClass('et_bloom_warn_field');
					} else {
						$this_wrapper.addClass('et_bloom_warn_field');
					}

					is_valid = false;
				}

				if ('email' === field_type) {
					// remove trailing/leading spaces and convert email to lowercase
					var processed_email = this_val.trim().toLowerCase();
					var is_valid_email = et_email_reg_html5.test(processed_email);

					if ('' !== processed_email && !is_valid_email) {
						$this_el.addClass('et_bloom_warn_field');
						is_valid = false;
					}
				}
			} );

			if (! is_valid) {
				return;
			}

			if ('' == email) {
				this_form.find('.et_bloom_subscribe_email input').addClass('et_bloom_warn_field');
			} else {
				ip_address = ip_address ? 'true' : 'false';
				$subscribe_data = JSON.stringify({ 'list_id' : list_id, 'account_name' : account_name, 'service' : service, 'name' : name, 'email' : email, 'page_id' : page_id, 'optin_id' : optin_id, 'last_name' : last_name, 'ip_address': ip_address });
				$.ajax({
					type: 'POST',
					dataType: 'json',
					url: bloomSettings.ajaxurl,
					data: {
						action : 'bloom_subscribe',
						subscribe_data_array : $subscribe_data,
						custom_fields: custom_fields,
						subscribe_nonce : bloomSettings.subscribe_nonce
					},
					beforeSend: function(data) {
						this_button.addClass('et_bloom_button_text_loading');
						this_button.find('.et_bloom_subscribe_loader').css({ 'display' : 'block' });
					},
					success: function(data) {
						this_button.removeClass('et_bloom_button_text_loading');
						this_button.find('.et_bloom_subscribe_loader').css({ 'display' : 'none' });
						if (data) {
							if ('' !== current_container && (data.success || 'Invalid email' !== data.error)) {
								unlock_content(current_container, container_id, locked_page_id, locked_optin_id);
							} else {
								if (data.error) {
									this_form.find('.et_bloom_error_message').remove();
									this_form.prepend('<h2 class="et_bloom_error_message">' + data.error + '</h2>');
									this_form.parent().parent().find('.et_bloom_form_header').addClass('et_bloom_with_error');
								}
								if (data.success && '' == current_container) {
									set_cookie(365, 'et_bloom_subscribed_to_' + optin_id + list_id + '=true');

									if ('' === success_action || '' === success_action_url) {
									  this_form.parent().find('.et_bloom_success_message').addClass('et_bloom_animate_message');
									  this_form.parent().find('.et_bloom_success_container').addClass('et_bloom_animate_success');
									  this_form.remove();
									} else {
										window.location = success_action_url;
									}

									// auto close popup if enabled
									if (is_popup && $popup_container.hasClass('et_bloom_auto_close')) {
										setTimeout(function() {
											perform_popup_closing($popup_container);
										},1400);
									}
								}
							}

							if (is_popup) {
								define_popup_position($popup_container, false, 50);
							}
						}
					}
				});
			}
		}

		$( 'body' ).on( 'click', '.et_bloom_custom_html_form input[type="submit"], .et_bloom_custom_html_form button[type="submit"]', function() {
			var this_button = $( this ),
				form_container = this_button.closest( '.et_bloom_custom_html_form' );

			update_stats_table( 'con', form_container );
		} );

		$( window ).resize( function(){
			if ( $( '.et_bloom_resize' ).length ) {
				$( '.et_bloom_resize' ).each( function() {
					define_popup_position( $( this ), false, 0 );
				});
			}
		});
	});
})(jQuery)
