# spryte
New website for Spryte

## Newsletter signups → Google Sheets

This site includes a simple email signup form that posts to a Google Apps Script web app which appends rows to a Google Sheet.

Setup steps:

1. Create a new Google Sheet (any name is fine).
2. In the sheet, go to Extensions → Apps Script and paste a doPost handler similar to the one below.
3. Deploy → Manage deployments → New deployment → Type: Web app
	 - Execute as: Me
	 - Who has access: Anyone with the link
	 - Deploy and copy the Web app URL
4. In `index.html`, set the `APPS_SCRIPT_URL` constant to your Web app URL.

Apps Script example:

function doPost(e) {
	try {
		var body = e && e.postData && e.postData.contents ? JSON.parse(e.postData.contents) : {};
		var email = (body.email || '').trim();
		if (!email) return json({ ok: false, error: 'Missing email' });
		var ss = SpreadsheetApp.getActive();
		var sheet = ss.getSheetByName('Sheet1') || ss.insertSheet('Sheet1');
		if (sheet.getLastRow() === 0) sheet.appendRow(['Timestamp', 'Email', 'Source']);
		sheet.appendRow([new Date(), email, 'website']);
		return json({ ok: true });
	} catch (err) {
		return json({ ok: false, error: String(err) });
	}
}
function json(obj) {
	return ContentService.createTextOutput(JSON.stringify(obj))
		.setMimeType(ContentService.MimeType.JSON);
}

Notes:

- The frontend sends Content-Type: text/plain with a JSON body in no-cors mode (opaque response) to avoid CORS preflight and match how the script JSON.parse()s e.postData.contents.
- Because of no-cors, the page can't read the response; it shows a success message optimistically after the request is sent.
- A simple honeypot field is included to deter basic bots.
