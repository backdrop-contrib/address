# Address

Provides a generic Form API element `#type = 'address'` that renders a dynamic, country-aware address form by reusing the Addressfield module’s format handlers.

- Module: Address
- Requires: addressfield
- Test page: /address/test

## Installation
- Enable the Address module (ensure the Addressfield module is enabled).
- Clear caches.

## Quick start (Form API)
Add this element to any Backdrop form:
```php
 $form['address'] = array(
    '#type' => 'address',
    '#title' => t('Address'),
    '#format_handlers' => array('name_full', 'organisation', 'address'),
    '#default_value' => array(
      'country' => 'CA',
      'organisation_name' => 'AltaGrade',
      'locality' => 'Toronto',
      'postal_code' => 'M5A 1A1',
      'thoroughfare' => '123 Main St',
      'premise' => 'Apt 4B',
      'first_name' => 'John',
      'last_name' => 'Doe',
      'administrative_area' => 'ON',
    ),
    '#available_countries' => array(),
    '#default_country' => 'CA',
    '#wrapper_id' => 'address-test-wrapper',
  );
```

On submit, the nested values are available under `$form_state['values']['address']`.

Common keys include:
- country, administrative_area, locality, postal_code
- thoroughfare (street), premise (apt/suite)
- organisation_name
- name_line (if a name handler is enabled)

Exact keys depend on enabled handlers and selected country’s format.

## Options
- `#format_handlers`: Controls which parts render, using Addressfield handlers. Examples: `name_full`, `organisation`, `address`. Order matters.
- `#available_countries`: Limit selectable countries (array of ISO codes).
- `#default_country`: Preselect a country when no value exists.
- `#default_value`: Prefill address parts (associative array).
- `#wrapper_id`: Optional stable DOM id wrapper for AJAX/theming.

## Test form
The module exposes a simple test page at `/address/test` showing the element in action and logging submitted values.

## How it works (internals)
- Element definition: `hook_element_info()` registers `#type = 'address'` with:
  - `#theme_wrappers` => `fieldset` for familiar UI.
  - `#process` => `address_element_process()` builds the inner controls.
  - `#input` is FALSE on the wrapper; children are the input elements.
- Process callback:
  - Builds a minimal fake Field/Instance context compatible with Addressfield.
  - Resolves defaults via `addressfield_default_values()`.
  - Builds the inner form with `addressfield_generate($address, $handlers, $context)`.
  - Attaches the generated structure under a single child (`$element['format']`) and re-parents it so submitted values post as `address[...]`.
  - Sanitizes wrapper metadata to avoid “Array to string conversion” warnings in `theme_fieldset()`.
- Alter point: `backdrop_alter('addressfield_handlers', ...)` lets other modules adjust the handlers list at build time.

## Available handlers
- `name_oneline`
  - Title: Show a Full Name field
  - Type: name
  - Weight: 0
  - Callback: addressfield_format_name_oneline_generate
  - File: formats/name-oneline.inc
- `name_full`
  - Title: Show First & Last name fields
  - Type: name
  - Weight: 0
  - Callback: addressfield_format_name_full_generate
  - File: formats/name-full.inc
- `organisation`
  - Title: Show Company/Organization field
  - Type: organisation
  - Weight: -10
  - Callback: addressfield_format_organisation_generate
  - File: formats/organisation.inc
- `address`
  - Title: Show the address form (country-specific)
  - Type: address
  - Weight: -100
  - Callback: addressfield_format_address_generate
  - File: formats/address.inc
- `address_hide_country`
  - Title: Hide country field (when only one is available)
  - Type: address
  - Weight: -90
  - Callback: addressfield_format_address_hide_country
  - File: formats/address-hide-country.inc
- `address_hide_postal_code`
  - Title: Hide postal code field (Zip)
  - Type: address
  - Weight: -85
  - Callback: addressfield_format_address_hide_postal_code
  - File: formats/address-hide-postal-code.inc
- `address_hide_administrative_area`
  - Title: Hide administrative area field (State/Province)
  - Type: address
  - Weight: -84
  - Callback: addressfield_format_address_hide_administrative_area
  - File: formats/address-hide-administrative-area.inc
- `address_hide_locality`
  - Title: Hide locality field (City)
  - Type: address
  - Weight: -84
  - Callback: addressfield_format_address_hide_locality
  - File: formats/address-hide-locality.inc
- `address_hide_street`
  - Title: Hide 1st street address field (Address/locality)
  - Type: address
  - Weight: -83
  - Callback: addressfield_format_address_hide_street
  - File: formats/address-hide-street.inc
- `address_hide_street_two`
  - Title: Hide 2nd street address field (Address 2/premise)
  - Type: address
  - Weight: -82
  - Callback: addressfield_format_address_hide_premise
  - File: formats/address-hide-premise.inc
- `address_optional`
  - Title: Make all fields optional (No validation - unsuitable for postal purposes)
  - Type: address
  - Weight: 100
  - Callback: addressfield_format_address_optional
  - File: formats/address-optional.inc

## Notes and limitations
- This module provides a Form API element, not a field storage type. Persist values yourself or map them into fields.
- Available parts and validation vary by country and selected handlers.
- Clear caches after changing handler availability or formats.

## Issues

Bugs and feature requests should be reported in the issue queue:
https://github.com/backdrop-contrib/bee/issues.

## Current Maintainers

- [Alan Mels](https://github.com/alanmels) - [AltaGrade](https://www.altagrade.com)
- Collaboration and co-maintainers welcome!

## License

This project is GPL v2 software.
See the LICENSE.txt file in this directory for complete text.

