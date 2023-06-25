---
title: Download CSV
parent: PHP
nav_order: 1
---

# Download Data as CSV

Send some data to the client as a CSV file attachment using simple, built-in functions within PHP.

Clearing all current output buffers before writing is useful when responding within a WordPress context such as a wp-admin page or WP REST API endpoint. It ensures only the CSV values are written to the resulting file attachment, which a client's web browser will automatically prompt for downloading to the client's machine.

```php
// Default file name for download.
$csv_file_name = date( 'Y-m-d' ) . '_values.csv';

// Define rows of data.
$csv_file_content = array(
	array( 'a', 'b', 'c' ),
	array( 1, 2, 3 ),
	array( 'Los Angelos', 'New York', 'Chicago' ),
);

// Tell client to download the contents as a file.
header( 'Content-Type: text/csv' );
header( 'Content-Disposition: attachment; filename="' . $csv_file_name . '"' );

// Clear all current output buffers.
while ( 0 !== ob_get_level() ) {
	ob_end_clean();
}

// Write the content as CSV.
$f = fopen( 'php://output', 'w' );
foreach ( $csv_file_content as &$line ) {
	fputcsv( $f, $line );
}

// All done!
exit;

```

