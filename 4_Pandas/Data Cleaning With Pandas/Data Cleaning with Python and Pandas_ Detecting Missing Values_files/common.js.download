(function($){
	$(document).ready( function(){
		var user_agent = navigator.userAgent;
		var is_opera_edge;
		var browser = user_agent.match(/(opera|chrome|safari|firefox|msie|trident(?=\/))/i) || [];
		var browser_name = '';
		var browser_class = '';

		if ( /trident/i.test( browser[0] ) ) {
			browser_name = 'ie';
		} else if ( browser[0] === 'Chrome' ) {
			is_opera_edge = user_agent.match(/\b(OPR|Edge)/);

			if ( is_opera_edge !== null ) {
				browser_name = is_opera_edge[0].replace('OPR', 'opera');
			}
		}

		// use navigator.appName as browser name if we were unable to get it from user_agent
		if ( '' === browser_name ) {
			if ('standalone' in window.navigator && !window.navigator.standalone) {
				browser_name = 'uiwebview';
			} else {
				browser_name = browser[0] && '' !== browser[0] ? browser[0] : navigator.appName;
			}
		}

		browser_name = browser_name.toLowerCase();

		// convert browser name to class. Some classes do not match the browser name
		switch( browser_name ) {
			case 'msie' :
				browser_class = 'ie';
				break;
			case 'firefox' :
				browser_class = 'gecko';
				break;
			default :
				browser_class = browser_name;
				break;
		}

		// add `iphone` class if browsing from iphone
		if ( user_agent.match(/iPhone/) ) {
			browser_class += ' iphone';
		}

		$( 'body' ).addClass( browser_class );
	});
})(jQuery);
