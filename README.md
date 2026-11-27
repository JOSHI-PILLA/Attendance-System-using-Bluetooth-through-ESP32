ATTENDANCE-ESP32 SYSTEM
=======================
IoT-based Bluetooth attendance with a Wi-Fi dashboard, built on a single ESP32.

Students mark their own attendance from their phones over Bluetooth. The
professor watches it update live on a password-protected web dashboard served
by the same board. No internet, router, or external server is needed.


FEATURES
--------
- Self-marking attendance over Bluetooth: a student sends a short personal code
  from any Bluetooth terminal app, so no custom app is needed.
- Own Wi-Fi hotspot: the ESP32 creates its own network, so it works in any
  classroom without internet.
- Password-protected professor dashboard with session-cookie login.
- Live summary: total, present, and absent counts update without refreshing.
- Manual control: add or delete students, and mark anyone present or absent.
- Search the report table to find a student quickly.
- CSV export of the attendance report.
- "New session (reset)" button to start a fresh class period.
- Flash storage: attendance is saved to the ESP32's internal flash, so a power
  cut does not lose records.
- Optional device-lock: stops one phone from marking more than one student
  present.


HOW IT WORKS
------------
The ESP32 acts as a single hub with two independent wireless paths:

  Student path   : Bluetooth (classic SPP)      -> short text codes
  Professor path : Wi-Fi (access point + HTTP)  -> dashboard page and data

Both paths read and write the same student list, which is saved to flash after
every change.

  1. Input      : A student's phone connects over Bluetooth and sends a code.
  2. Processing : The ESP32 checks the code against its student list and marks
                  the student present with a timestamp.
  3. Storage    : The list is saved to flash using the Preferences library.
  4. Output     : The dashboard shows the update, and the student's terminal
                  receives an acknowledgement.

Classic Bluetooth connects to one phone at a time, so the ESP32 disconnects
each phone after a successful mark so the next student can connect.


HARDWARE REQUIRED
-----------------
- ESP32 Dev Module    : Bluetooth input, Wi-Fi access point, web server, flash
- USB cable / power bank : Powers the board (use a data-capable cable to program)
- Students' smartphones  : Send the attendance code over Bluetooth
- Professor's laptop or phone : Views the dashboard in a browser

There is no external wiring. Every connection is wireless, and the board only
needs USB power.


SOFTWARE REQUIRED
-----------------
- IDE              : Arduino IDE with the ESP32 board package installed
- Board            : ESP32 Dev Module
- Partition scheme : Huge APP (3MB No OTA/1MB SPIFFS)
- Libraries        : BluetoothSerial, WiFi, WebServer, Preferences
                     (all bundled with the ESP32 board package)
- Front-end        : Single-file HTML/CSS/JavaScript dashboard served from
                     flash as a C header file (webpage.h)
- Student app      : Any Bluetooth terminal app (e.g. Serial Bluetooth Terminal)

Default names and address:
- Bluetooth device name : Attendance-ESP32
- Wi-Fi hotspot name    : Attendance-ESP32-Dash
- Dashboard address     : 192.168.4.1

IMPORTANT: The default partition scheme is too small for the combined Bluetooth
and Wi-Fi firmware. Select "Huge APP" under Tools > Partition Scheme before
uploading, or the upload will fail.


GETTING STARTED
---------------
1. Install the ESP32 board package in the Arduino IDE (Boards Manager).
2. Open the sketch and make sure webpage.h is in the same folder.
3. Set the professor login (username and password) in the sketch.
4. Select the board and partition scheme:
     Board            : ESP32 Dev Module
     Partition Scheme : Huge APP (3MB No OTA/1MB SPIFFS)
5. Connect the ESP32 with a data-capable USB cable and click Upload.
6. Power the board from a laptop or a power bank.


USAGE
-----
Run one attendance session like this:

1. Power the ESP32.
2. On the professor's device, join the Wi-Fi hotspot "Attendance-ESP32-Dash"
   and open 192.168.4.1 in a browser.
3. Log in with the professor's username and password.
4. Add each student's code and full name using the Add student form, or check
   that the list is already correct.
5. Students pair their phones with "Attendance-ESP32" and send their code one
   after another from a Bluetooth terminal app.
6. Watch the dashboard update live. Use Mark present / Mark absent for any
   manual correction.
7. At the end of the class, download the report as a CSV file, then press
   "New session (reset)" before the next class.


MARKING LOGIC
-------------
Situation                                   Result                        Saved to flash   Phone
------------------------------------------  ----------------------------  ---------------  ------------
Valid code, student not yet present         Marked Present + timestamp    Yes              Disconnected
Code not in the student list                No marking made               No               -
Student already marked present              Attendance unchanged          -                -
Device-lock on, phone already used for      Not marked                    No               -
  another student
Professor marks present/absent on the       Status changed; "(manual)"    Yes              -
  dashboard                                 shown in Marked-at column


API ENDPOINTS
-------------
The dashboard uses these REST-style endpoints:

  /api/students   Read the current student list and attendance status
  /api/add        Add a new student (code and full name)
  /api/mark       Manually mark a student present or absent
  /api/delete     Remove a student from the list
  /api/reset      Start a new session by clearing all attendance
  /api/download   Download the attendance report as a CSV file


TEST RESULTS
------------
The system was tested with the ESP32 powered from a laptop and two phones
acting as student devices.

  Test                          Terminal response                Dashboard
  ----------------------------  -------------------------------  --------------------
  Indra's code sent             OK Indra marked PRESENT          1 Present / 4 Absent
  Further students' codes sent  READY, then a confirmation       3 Present / 2 Absent
                                per code

Observations:
- Each accepted code was acknowledged on the phone and shown on the dashboard
  without any manual refresh.
- The phone was disconnected after each successful mark, freeing the Bluetooth
  link for the next student.
- The summary counters always matched the rows of the report table.


LIMITATIONS
-----------
- Without device-lock, a student could mark a classmate present if given their
  code.
- Classic Bluetooth connects to one phone at a time, so students mark
  attendance one after another.
- Student codes and the professor password are stored in plain text on the
  device. This is fine for a classroom prototype but not for production.
- With no router connected, the board has no real-time clock, so "Marked at"
  shows device uptime in seconds.


FUTURE ENHANCEMENTS
-------------------
- Switch to Bluetooth Low Energy (BLE) so several students can mark attendance
  at the same time.
- Add face or fingerprint verification as a second factor.
- Sync to the cloud (e.g. Firebase or Google Sheets) when internet is
  available.
- Send email or SMS alerts to parents when a student is marked absent.
- Generate attendance percentage reports over a semester.


TEAM
----
Project by:
- P. Joshi
