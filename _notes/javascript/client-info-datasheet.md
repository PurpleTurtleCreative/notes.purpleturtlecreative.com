---
title: Client Info Datasheet
parent: JavaScript
nav_order: 1
---

# Client Info Datasheet

The following snippet is helpful for hosting a webpage that clients or users may visit to then send a screenshot with diagnostic information about their web browsing client.

This was originally used to debug internationalization features in a ReactJS application.

```html
<script type="text/javascript">

const container = document.getElementById('client-datasheet');
const now = new Date();

try {

	const localeNewYork = 'en-US';
	const optionsNewYork = {
		dateStyle: 'full',
		timeStyle: 'long',
		timeZone: 'America/New_York'
	};

	const intlNewYork = (new Intl.DateTimeFormat(localeNewYork, optionsNewYork)).resolvedOptions();
	const intlClient = (new Intl.DateTimeFormat()).resolvedOptions();

	const nowNewYorkString = now.toLocaleString(localeNewYork, optionsNewYork);
	const nowLocaleString = now.toLocaleString(
		intlClient.locale,
		{
			dateStyle: 'full',
			timeStyle: 'long'
		}
	);
	const nowLocaleEnglishString = now.toLocaleString(
		localeNewYork,
		{
			dateStyle: 'full',
			timeStyle: 'long'
		}
	);

	const nowNewYorkTimeZoneOffsetString = (new Intl.DateTimeFormat(
		localeNewYork,
		{
			timeZone: optionsNewYork.timeZone,
			timeZoneName: 'longOffset'
		}
	)).formatToParts(now).find(part => 'timeZoneName' === part.type).value;
	const nowClientTimeZoneOffsetString = (new Intl.DateTimeFormat(
		localeNewYork,
		{
			timeZoneName: 'longOffset'
		}
	)).formatToParts(now).find(part => 'timeZoneName' === part.type).value;

	container.innerHTML = `
	<figure class="wp-block-table is-style-stripes">
		<table>
			<thead>
				<tr>
					<th></th>
					<th>Reference Locale</th>
					<th>Your Locale</th>
				</tr>
			</thead>
			<tbody>
				<tr>
					<th>Locale Code</th>
					<td>${intlNewYork.locale}</td>
					<td>${intlClient.locale}</td>
				</tr>
				<tr>
					<th>Calendar</th>
					<td>${intlNewYork.calendar}</td>
					<td>${intlClient.calendar}</td>
				</tr>
				<tr>
					<th>Numbering System</th>
					<td>${intlNewYork.numberingSystem}</td>
					<td>${intlClient.numberingSystem}</td>
				</tr>
				<tr>
					<th>Time Zone</th>
					<td>${intlNewYork.timeZone}</td>
					<td>${intlClient.timeZone}</td>
				</tr>
				<tr>
					<th>Time Zone Offset</th>
					<td>${nowNewYorkTimeZoneOffsetString}</td>
					<td>${nowClientTimeZoneOffsetString}</td>
				</tr>
				<tr>
					<th>Locale String</th>
					<td>${nowNewYorkString}</td>
					<td>${nowLocaleString}</td>
				</tr>
				<tr>
					<th>Locale String (en-US)</th>
					<td>${nowNewYorkString}</td>
					<td>${nowLocaleEnglishString}</td>
				</tr>
			</tbody>
		</table>
	</figure>
	`;
} catch (err) {
	container.innerHTML = `<p class="banner banner-danger"><strong>ERROR:</strong> ${err}</p>`;
}
</script>
<style type="text/css">
	table tbody th {
		text-align: right;
	}
</style>
```
