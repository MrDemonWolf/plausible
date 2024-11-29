# Important changes to Plausible WordPress plugin to allow self-hosted Plausible to Proxy

I have made the following changes to the Plausible WordPress plugin to allow self-hosted Plausible to Proxy.

# src/Proxy.php

```php
/**
	 * Formats and sends $request to the Plausible API.
	 *
	 * @return array|WP_Error
	 */
	public function send_event( $request ) {
		$params = $request->get_body();

		$ip  = $this->get_user_ip_address();

        // Add this line if you want to allow self-hosted Plausible to Proxy
		$url = 'https://plausible.io/api/event';
		if ( defined( 'PLAUSIBLE_SELF_HOSTED_DOMAIN' ) ) {
			$url = "https://" . PLAUSIBLE_SELF_HOSTED_DOMAIN . "/api/event/";
		};

		$ua  = ! empty ( $_SERVER[ 'HTTP_USER_AGENT' ] ) ? wp_kses( $_SERVER[ 'HTTP_USER_AGENT' ], 'strip' ) : '';

		return wp_remote_post(
			$url,
			[
				'user-agent' => $ua,
				'headers'    => [
					'X-Forwarded-For' => $ip,
					'Content-Type'    => 'application/json',
				],
				'body'       => wp_kses_no_null( $params ),
			]
		);
	}
```

## src/admin/settings/Page.php

    ```php
    [
    				'label'  => esc_html__( 'Bypass ad blockers', 'plausible-analytics' ),
    				'slug'   => 'bypass_ad_blockers',
    				'type'   => 'group',
    				'desc'   => sprintf(
    					wp_kses(
    						__(
    							'Concerned about ad blockers? You can run the Plausible script as a first-party connection from your domain name to count visitors who use ad blockers. The proxy uses WordPress\' API with a randomly generated endpoint, starting with <code>%1$s</code> and %2$s. <a href="%3$s" target="_blank">Learn more &raquo;</a>',
    							'plausible-analytics'
    						),
    						wp_kses_allowed_html( 'post' )
    					),
    					get_site_url( null, rest_get_url_prefix() ),
    					empty(
    					Helpers::get_settings()[ 'proxy_enabled' ]
    					) ? 'a random directory/file for storing the JS file' : 'a JS file, called <code>' . str_replace(
    							ABSPATH,
    							'',
    							Helpers::get_proxy_resource( 'cache_dir' ) . Helpers::get_proxy_resource(
    								'file_alias'
    							) . '.js</code>'
    						),
    					'https://plausible.io/wordpress-analytics-plugin#how-to-enable-a-proxy-to-get-more-accurate-stats'
    				),
    				'toggle' => '',
    				'fields' => [
    					[
    						'label'    => esc_html__( 'Enable proxy', 'plausible-analytics' ),
    						'slug'     => 'proxy_enabled',
    						'type'     => 'checkbox',
    						'value'    => 'on',
                            // Remove this line if you want to allow self-hosted Plausible to Proxy
    						'disabled' => ! empty( Helpers::get_settings()[ 'self_hosted_domain' ] ),
    					],
    				],
    			],

```

src/admin/settings/Hooks.php
```php
	public function proxy_warning() {
        // Remove this line if you want to allow self-hosted Plausible to Proxy
		if ( ! empty( Helpers::get_settings()[ 'self_hosted_domain' ] ) ) {
			$this->option_na_in_ce();
            // Add this line if you want to allow self-hosted Plausible to Proxy
            echo sprintf(
				wp_kses(
					__(
						'After enabling this option, please check your Plausible dashboard to make sure stats are being recorded. Are stats not being recorded? Check github issues for <a href="%s" target="_blank">known issues</a>. We\'re here to help!',
						'plausible-analytics'
					),
					'post'
				),
				'https://github.com/plausible/analytics/issues'
			);
		} else {
			echo sprintf(
				wp_kses(
					__(
						'After enabling this option, please check your Plausible dashboard to make sure stats are being recorded. Are stats not being recorded? Do <a href="%s" target="_blank">reach out to us</a>. We\'re here to help!',
						'plausible-analytics'
					),
					'post'
				),
				'https://plausible.io/contact'
			);
		}
	}
```

