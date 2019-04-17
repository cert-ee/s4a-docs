# Yara rules and Wise feeds

## Managing Yara rules and Wise

This is is done via Central -> Feeds interface:
1. Administrator adds feed from Web UI (will create file as well, if not preset) and adds appropriate tag(s).
2. Feed file can be updated with necessary data.
3. System updates data file checksums for Detectors to be able to detect changes.

## Detector updates

Detector has only means to disable and enable Yara and Wise via Settings.
1. Detector requests update providing current list of feeds.
2. Central compares list with local list (with matching tags) and checksums and provides updates or new feeds available for Detector.
3. Detector gets updates and reloads necessary services (or, if supported, services will update configuration on runtime)
