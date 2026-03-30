# Fastmail CalDAV Integration (Calendar)

## What it does

Query today/tomorrow calendar events for daily check-ins. Supports recurring event expansion.

## Auth

Basic auth with app password:

```
-u '[EMAIL]:[APP_PASSWORD]'
```

Token location: `.env` → `FASTMAIL_APP_PASSWORD`

## Calendar Discovery

List all calendars:

```bash
curl -s -u '[EMAIL]:[APP_PASSWORD]' \
  'https://caldav.fastmail.com/dav/calendars/user/[EMAIL]/' \
  -H 'Depth: 1' -X PROPFIND \
  -H 'Content-Type: application/xml' \
  -d '<?xml version="1.0" encoding="UTF-8"?>
<d:propfind xmlns:d="DAV:"><d:prop><d:displayname/><d:resourcetype/></d:prop></d:propfind>'
```

Parse the response for `calendar` resource types to get calendar URLs and names. Cache the calendar URL — it doesn't change.

## Query Events (Date Range)

```bash
curl -s -u '[EMAIL]:[APP_PASSWORD]' \
  'https://caldav.fastmail.com/dav/calendars/user/[EMAIL]/[CALENDAR_ID]/' \
  -H 'Depth: 1' -X REPORT \
  -H 'Content-Type: application/xml' \
  -d '<?xml version="1.0" encoding="UTF-8"?>
<c:calendar-query xmlns:d="DAV:" xmlns:c="urn:ietf:params:xml:ns:caldav">
  <d:prop><d:getetag/><c:calendar-data/></d:prop>
  <c:filter>
    <c:comp-filter name="VCALENDAR">
      <c:comp-filter name="VEVENT">
        <c:time-range start="[YYYYMMDD]T000000Z" end="[YYYYMMDD]T000000Z"/>
      </c:comp-filter>
    </c:comp-filter>
  </c:filter>
</c:calendar-query>'
```

For daily check-ins, set the range to cover today and tomorrow (2 days).

## Parsing the Response

The response contains iCalendar (`.ics`) data. Extract these fields per event:

- `SUMMARY` — event title
- `DTSTART` — start time
- `DTEND` — end time
- `RRULE` — recurrence rule (if present, this is a recurring event)

## Gotchas

- CalDAV returns raw iCalendar format embedded in XML — parse both layers.
- Recurring events are returned with their `RRULE`, not expanded instances. Check if the recurrence pattern matches the queried date range.
- App password is different from your main password. Generate one in Fastmail settings under Privacy & Security.
