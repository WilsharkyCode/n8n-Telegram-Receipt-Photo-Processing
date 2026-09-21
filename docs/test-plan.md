# Test Plan

Manual test cases for the Telegram receipt logger.
Fill in the **Result** column as you run each case. Some cases document behavior that is known and not yet handled (marked **Current behavior**), so record what actually happens.

**Environment**

- n8n version: `___`
- Run mode: `___` (local + tunnel / Docker / cloud)
- Telegram bot: a dedicated test bot
- Test sheet and Drive: dedicated test copies, not your real records

## How to run

- Activate the workflow, then send photos to the bot from your Telegram account.
- After each test, check three places: the `Receipts` tab, the Drive folder, and the bot's reply. Also open the **Executions** tab in n8n to see whether the run succeeded.
- For failure tests, deactivate or break one piece (an API key or credential), send a photo, and restore it afterward.

## A. Happy path

| # | Case | Setup | Expected result | Result |
|---|---|---|---|---|
| 1 | Clear receipt | Send a sharp, well-lit receipt photo | One new row with merchant, amount, currency and transactionDate matching the receipt. Photo saved in a `YYYY-MM-DD` folder. Confirmation reply received. | ☐ Pass ☐ Fail |
| 2 | All columns filled | Inspect the row from test 1 | All eight columns have values, including submissionDate, folderName, fileName and chatId. | ☐ Pass ☐ Fail |
| 3 | Reply matches the sheet | Compare the Telegram reply with the row | Merchant, amount and currency are identical. | ☐ Pass ☐ Fail |
| 4 | Saved image is the original | Open the uploaded file in Drive | It is the same photo you sent, readable and not corrupted. | ☐ Pass ☐ Fail |
| 5 | Long receipt with many items | Send a receipt with several line items, a subtotal and a total | The amount logged is the final total, not the subtotal or a line item. | ☐ Pass ☐ Fail |
| 6 | Foreign currency | Send a receipt in another currency (for example USD) | Currency is extracted correctly and not assumed to be PHP. | ☐ Pass ☐ Fail |
| 7 | Amount formats | Send receipts with `1,234.50` style and, if you have one, `1.234,50` style amounts | Amount is read correctly in both. Note how each is stored. | ☐ Pass ☐ Fail |

## B. Folders and filenames

| # | Case | Setup | Expected result | Result |
|---|---|---|---|---|
| 8 | New day, new folder | Send a receipt on a day with no folder yet | The date folder is created in the Drive root and the photo is inside. | ☐ Pass ☐ Fail |
| 9 | Second receipt, same day | Send another receipt after the first | The existing folder is reused. No duplicate folder. | ☐ Pass ☐ Fail |
| 10 | Filename collision | Note the filenames from test 9 | **Current behavior:** both files have the same name (`receipt_<date>_<chatId>.jpg`). Both are kept in Drive. | ☐ Pass ☐ Fail |
| 11 | Two receipts at once | On a day with no folder, send two photos within a second or two | Check whether one or two date folders are created. **Current behavior:** may create duplicates. | ☐ Pass ☐ Fail |
| 12 | Date boundary | Send a receipt between 00:00 and 08:00 Philippine time | **Current behavior:** the folder and submissionDate use the UTC date, which is the previous day locally. | ☐ Pass ☐ Fail |

## C. Extraction quality and odd inputs

| # | Case | Setup | Expected result | Result |
|---|---|---|---|---|
| 13 | Receipt with no visible date | Send a receipt where the date is missing or cut off | The run still completes. transactionDate is blank or wrong. Record which. | ☐ Pass ☐ Fail |
| 14 | Blurry or dark photo | Send a low-quality photo | Record the outcome: wrong values, blank values, or an error. | ☐ Pass ☐ Fail |
| 15 | Non-receipt photo | Send a photo that isn't a receipt | Record what gets logged. Check that nothing crashes the workflow. | ☐ Pass ☐ Fail |
| 16 | Consistency of keys | Send five different receipts | Every row has values in merchant, amount, currency and transactionDate. No column is consistently empty (this would mean the model used different key names). | ☐ Pass ☐ Fail |

## D. Invalid input and access

| # | Case | Setup | Expected result | Result |
|---|---|---|---|---|
| 17 | Text message | Send plain text to the bot | **Current behavior:** the execution fails with "No photo found". No row, no upload, no reply. | ☐ Pass ☐ Fail |
| 18 | Photo sent as a file | Attach the image as a file/document, not as a photo | **Current behavior:** same failure as test 17. | ☐ Pass ☐ Fail |
| 19 | Message from another account | Send a receipt from a different Telegram account | **Current behavior:** it is processed like any other, because there is no chat ID check. | ☐ Pass ☐ Fail |

## E. Failure handling

| # | Case | Setup | Expected result | Result |
|---|---|---|---|---|
| 20 | Gemini fails | Temporarily set an invalid `GEMINI_API_KEY` and send a receipt | Execution fails. No sheet row, no upload, no reply. | ☐ Pass ☐ Fail |
| 21 | Drive fails | Disconnect or break the Google Drive credential and send a receipt | **Current behavior:** the sheet row is written, but there is no image in Drive and no reply. | ☐ Pass ☐ Fail |
| 22 | Sheet fails | Rename the `Receipts` tab or change a header, then send a receipt | Execution fails. Note whether Drive upload and reply are skipped. | ☐ Pass ☐ Fail |

## F. Setup

| # | Case | Setup | Expected result | Result |
|---|---|---|---|---|
| 23 | Webhook registration | Activate the workflow with a valid public HTTPS `WEBHOOK_URL` | The Telegram trigger activates and the bot responds to photos. | ☐ Pass ☐ Fail |
| 24 | Address change | Restart your tunnel (if you use a quick tunnel) and send a photo | The bot stops responding until `WEBHOOK_URL` is updated and the workflow is reactivated. | ☐ Pass ☐ Fail |

## Notes

<!-- Record failures, fixes and anything you'd change. This section shows how you debug. -->

| # | Issue found | Cause | Fix |
|---|---|---|---|
|   |   |   |   |
