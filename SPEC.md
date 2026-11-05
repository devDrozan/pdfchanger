# PDF Changer — feature specification

## First release

Users can photograph one or more document pages with the camera or import images from Photos or Files. Camera capture remains available without a network connection. Before creating a PDF, users can crop, rotate, remove, and reorder pages. The PDF places each image on an A4 page with margins, without stretching it.

Users can send completed PDFs from the Library to Merge PDFs. Sent PDFs appear in a persistent Recent list and are added to the current merge selection. Users can also add PDFs from Recent or choose PDFs from Files, then select at least two inputs, change their order, and combine all their pages into one PDF. The resulting page sequence follows the chosen file order and each source PDF's internal page order.

Completed PDFs appear in a local library. Users can preview, rename, delete, and send a completed PDF to Merge PDFs. Imported images and PDFs stay unchanged; temporary inputs do not become library items. The library and Recent merge list remain available after the app restarts.

Users can switch the app interface between English and Burmese at any time. The choice is saved on the device and remains selected after restarting the app. Switching languages does not alter PDFs, imported files, or names the user has entered.

On launch, the app shows its logo for at least three seconds while it prepares the library and saved language choice. It then fades smoothly into the main screen without showing an empty screen. The language selector is clearly labeled, aligned above the main navigation, and available from the main screen.

Core creation, merging, and browsing work without a network connection. The first release does not include accounts, cloud sync, searchable text, or editing a PDF after saving it.

## Acceptance criteria

- A user can photograph or import multiple pages, arrange and edit them, then save a readable A4 PDF.
- A user can send a library PDF to Merge PDFs without removing the original, select PDFs from Recent or Files, reorder two or more selected PDFs, and save a merged file with all pages in that order.
- A canceled camera capture or picker leaves the current work and library intact.
- Unreadable or protected PDF inputs, unusable images, and insufficient device storage show a clear error without adding an incomplete PDF to the library.
- Completed PDFs remain available after restarting the app, and can be previewed, renamed, and deleted. Recent merge entries survive restart and reflect library renames and deletions.
- A user can switch between English and Burmese on every screen; buttons, instructions, dialogs, progress messages, and common errors use the selected language. The choice persists after restart.
- On a fresh launch, the logo remains visible for at least three seconds and until the main screen is ready, then fades into the app. The language selector is easy to find and use in either language.
