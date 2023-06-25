---
title: Cards Grid Filter
parent: JavaScript
nav_order: 1
---

# Cards Grid Filter

A quick feature that clients commonly want is adding filter buttons to a large grid of items (often styled as "cards"). The following code dynamically figures out what filters apply to the current grid of items, renders the filter buttons, and controls their filtering affect. Additionally, this code was made to be compatible with an AJAX-loaded grid of items.

***Note that the grid of items must NOT be paginated for this to have the correct effect.*

```javascript
jQuery(function($){

	if ( false === document.body.classList.contains("type-place") ) {
		return; // Only run on body.type-place!
	}

	// DOM nodes.

	const $beers_tab = $('#listing_tab_beers');
	let $beers_tab_cards = $beers_tab.find('.tab-contents > div');

	$beer_options_container = $(`<div class="container ptf-beer-filter-options"></div>`);
	$beers_tab.prepend($beer_options_container);

	// Define beer type options manager.
	const beer_types = {

		beerCardsByType: {},

		beerFilterOptionButtons: [],

		registerBeerCard: function(beerType, $el) {
			if ( false === ( beerType in this.beerCardsByType ) || false === Array.isArray(this.beerCardsByType[ beerType ]) ) {
				this.beerCardsByType[ beerType ] = [];
			}
			this.beerCardsByType[ beerType ].push($el);
		},

		insertOptionButtons: function($target) {

			beerFilterOptionButtons = [];

			$beer_option_reset_button = $(`<button type="button" class="ptf-beer-filter ptf-beer-filter-reset active" value="view-all">View All (${$beers_tab_cards.length})</button>`);
			$beer_option_reset_button.on('click', this.handleBeerFilterOptionClick.bind(this));
			$target.append($beer_option_reset_button);
			this.beerFilterOptionButtons.push($beer_option_reset_button);

			for ( const beerType of Object.keys(this.beerCardsByType).sort() ) {
				$beer_option_button = $(`<button type="button" class="ptf-beer-filter" value="${beerType}">${beerType} (${this.beerCardsByType[ beerType ].length})</button>`);
				$beer_option_button.on('click', this.handleBeerFilterOptionClick.bind(this));
				$target.append($beer_option_button);
				this.beerFilterOptionButtons.push($beer_option_button);
			}
		},

		handleBeerFilterOptionClick: function(event) {

			const clicked_beer_type = event.target.value;

			// Remove active class.
			for ( const $beer_option_button of this.beerFilterOptionButtons ) {
				$beer_option_button.removeClass('active');
			}

			if ( clicked_beer_type in this.beerCardsByType ) {
				// Hide all beer cards.
				$beers_tab_cards.each((i, el) => { $(el).hide(); });
				// Show all matching beer cards.
				for ( const $beer_tab_card of this.beerCardsByType[ clicked_beer_type ] ) {
					$beer_tab_card.show();
				}
			} else {
				// Show all beer cards.
				$beers_tab_cards.each((i, el) => { $(el).show(); });
			}

			// Add active class to clicked filter.
			$(event.target).addClass('active');
		},

		handleAjaxComplete: function(event, jqXHR, ajaxOptions) {

			// Check if AJAX event is relevant and successful.
			if ( false === ajaxOptions.url.includes('mylisting-ajax=1&action=get_related_listings') || 200 !== jqXHR.status ) {
				return; // Abort. Unrelated event.
			}

			// Collect beer type tokens.
			this.beerCardsByType = {};
			$beers_tab_cards = $beers_tab.find('.tab-contents > div');
			$beers_tab_cards.each((i, el) => {
				const beer_type_text = el.querySelector('.beerPreviewTypebg').innerText.trim();
				const beer_type_text_tokens = beer_type_text.split(' ');
				const last_token = beer_type_text_tokens[ beer_type_text_tokens.length - 1 ];
				if ( 'ale' === last_token.toLowerCase() ) {
					// Check if "pale ale".
					if ( 'pale' === beer_type_text_tokens[ beer_type_text_tokens.length - 2 ].toLowerCase() ) {
						this.registerBeerCard('Pale Ale', $(el));
					} else {
						this.registerBeerCard(last_token, $(el));
					}
				} else {
					// Add to list.
					this.registerBeerCard(last_token, $(el));
				}
			});

			window.console.log(this);

			// Add beer type filter option buttons.
			$beer_options_container.html('');
			this.insertOptionButtons($beer_options_container);
		}
	};

	// Update DOM whenever AJAX events complete.
	$(document).ajaxComplete(beer_types.handleAjaxComplete.bind(beer_types));
});
```
