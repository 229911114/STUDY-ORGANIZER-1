I started creating the code by developing three foundational modules to take care of environment initializing, secure data and automated testing.

*The "setup.py" file:
-This created a script that uses pythons "pathlibs" to locate the application directory and automatically make a "data" directory.
-Configured the script to intialize default, empty JSON storage files("tasks.json", "notes.json", and "schedule.json") if they dont already exisr, preventing crashes.

*The "storage.py" file:
- Implemented a secure file I/O module handling data loading(load_data) and saving(save_data) for approved json files.
- built robust error management to catch missing files(FileNotFoundError) and malformed json(ValueError).
- added path traversal protection by enforcing an explicit list of allowed filenames(ALLOWED_FILES).

*The "test_storage.py" file:
-Developed a test case (TestStorage) using pythons buil-in "unittest" framework.
-wrote security and error-handling tests(test_invalid_filename and test_invalid_json) to ensure the storage gateway correct rejects paths and handles corrupted json data safely.

I even created the "main.py" file and added the base so everyone can connect their files and import functions.
 
