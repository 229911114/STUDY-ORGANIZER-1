# study-organizer-project-1
This is a study orginizer app built using CLI, loops, automation, files and scripts.
Its designed to help students keep their academic life orginized.
Instead of juggling multiple apps,sticky notes, or messy files, this app brings everything a studedent needs into one clean space.
-The core purpose is to provide a fast, reliable, and distraction-free way to manage schoolwork. Because it runs right in your command line, it's quick to open, doesn't require heavy software, and gives you total control over your data.

*The "setup.py" file is the app's preparation and safety check before it starts running.
  1-When you download or clone a project from GitHub, empty folders (like a data/     folder) usually aren't included because Git only tracks files, not empty          directories. If a user downloads your app and runs it immediately, the app        will try to read tasks.json or schedule.json from a data/ folder that             doesn't exist yet, causing the program to crash.
 2 -"set.py" solves this by automatically setting up the environment: it builds       the missing folder and creates empty, safe starter files so the app runs           smoothly right out of the box.
 3 -It uses "pathlib", this ensures the app always knows exactly where to look for    files, no matter where it is executed from.
  -This script checks if the data/ folder exists, and if it doesn't, it creates it.
  4-This script defines a list of required data files and populates them with a          basic, valid JSON structure.

*The "storage.py" file:
  1- this is the is the secure data bridge between your application and your computer's hard drive. Whenever the app needs to save a task, load a schedule, or read a note, it goes through storage.py.
 2 - Without a dedicated storage file, reading and writing files could get messy, cause bugs, or worse, corrupt your data if the computer loses power or the program crashes right in the middle of saving. storage.py was built to make data saving 100% safe and secure.
  3- To prevent someone ,or a bug from trying to read or write unauthorized files on your computer (like system files), storage.py uses an explicit whitelist:
 ALLOWED_FILES = {"tasks.json", "schedule.json", "notes.json"}

     def _validate_filename(filename):
    if filename not in ALLOWED_FILES:
        raise ValueError(f"Access denied: '{filename}' is not an allowed storage      file.")
    -If a file name isn't in the ALLOWED_FILES list, the program immediately          rejects it and stops execution.

  4-When loading data (load_data), things can sometimes go wrong—like a file getting accidentally deleted or filled with gibberish. The loader handles these edge cases smoothly:
         
    def load_data(filename):
    _validate_filename(filename)
    file_path = DATA_DIR / filename
    
    if not file_path.exists():
        return []  # Return an empty list if file doesn't exist yet

    try:
        with open(file_path, "r") as f:
            return json.load(f)
    except (json.JSONDecodeError, ValueError):
        return []  # Return an empty list if the JSON file is corrupted

  -Instead of crashing the entire app with a red error screen, it catches the error and safely returns an empty list [], letting the user keep working.   

  *The "test_storage.py" file:
    1-This is the automated testing suite.
    Instead of manually opening the app, typing tasks, and checking if files update correctly every single time you change code, this script runs automated checks in a fraction of a second. It uses Python’s built-in unittest framework to verify that your storage layer (storage.py) handles normal saving/loading, blocks security threats, and survives corrupted files without crashing.

  *The "task_manager.py" file:
    1-task_manager.py is the functional heart of the app's homework and assignment tracking. While storage.py handles how data is written to the hard drive, task_manager.py handles the business logic—what a task actually looks like, how new tasks are created, how they are updated, marked complete, or deleted.
    2-Instead of writing file handling code all over again, task_manager.py imports storage.py to fetch and store task lists cleanly:
            
      import storage
     FILENAME = "tasks.json"

    def get_all_tasks():
    # Load all tasks from storage
    return storage.load_data(FILENAME)
  -It keeps the code modular. The task manager only worries about tasks, while the storage layer worries about files.

  3-The function "add_task' makes it  add a homework assignment, the manager creates a structured dictionary (with details like an ID, description, and status) and saves it:
    def add_task(description):
    tasks = get_all_tasks()
    
    # Assign a unique ID based on existing tasks, or start at 1
    new_id = tasks[-1]["id"] + 1 if tasks else 1
    
    new_task = {
        "id": new_id,
        "description": description,
        "completed": False  # Tasks start as uncompleted by default
    }
    
    tasks.append(new_task)
    storage.save_data(FILENAME, tasks)
    print(f"Task {new_id} added successfully!")

  -It automatically assigns IDs so tasks don't get mixed up, and ensures every new task defaults to completed: False.

  4-the "complete_task" function checks off a task, the manager searches through the list by ID and flips its status:
  def complete_task(task_id):
    tasks = get_all_tasks()
    found = False
    
    for task in tasks:
        if task["id"] == task_id:
            task["completed"] = True
            found = True
            break
            
    if found:
        storage.save_data(FILENAME, tasks)
        print(f"Task {task_id} marked as complete!")
    else:
        print(f"Error: Task with ID {task_id} not found.")

  - It gives users interactive control to update their progress and automatically persists the change back to the disk.

*The "schedule_manager" file:
  1-Just like task_manager.py handles the homework, schedule_manager.py handles your time. It is responsible for managing your weekly study blocks, lecture times, or daily academic timetables.
  2-While exact details can vary based on the  implementation, a schedule manager typically handles three core things: viewing the  study blocks, adding new schedule entries, and removing or clearing old ones.

3 -Loading the weekly schedule , It pulls schedule data from schedule.json using the secure storage core:   
                       
          import storage

         FILENAME = "schedule.json"
        def get_schedule():
       # Load all scheduled study blocks from storage
       return storage.load_data(FILENAME)

- It ensures that every time you check the timetable, it reads the most up-to-date saved data from the  disk.

4- the function "add_schedule_mananger" plans a study session (for example: "Monday, 2:00 PM - Linear Algebra"), the manager structures that data and saves it:    
             
     def add_schedule_item(day, time_slot, subject):
    schedule = get_schedule()
    
    new_entry = {
        "day": day,         
        "time": time_slot,  
        "subject": subject  
    }
        schedule.append(new_entry)
    storage.save_data(FILENAME, schedule)
    print(f"Scheduled '{subject}' for {day} at {time_slot}!")

    
-It keeps the  time blocks organized in a clean dictionary format, making it easy to display them nicely in your terminal or pass them into system reports later.

 5- You can view all your scheduled blocks or remove entries when your weekly routine changes:
 def view_schedule():
    schedule = get_schedule()
    if not schedule:
        
        print("Your study schedule is currently empty.")
        return
        print("\n--- Weekly Study Schedule ---")
    for idx, item in enumerate(schedule, 1):
        print(f"{idx}. [{item['day']}] {item['time']} - {item['subject']}")
- It presents the  data in a human-readable format right inside the command line, so you always know what you're supposed to be studying next.       

 *The "notes_manger.py" file:
  1-While task_manager.py tracks the  to-dos and schedule_manager.py tracks the  time, notes_manager.py acts as your the notebook.
  2- Like the other managers, it relies on storage.py for saving data to notes.json. 
  3-It pulls your stored notes from the JSON file:
    
      import storage
      FILENAME = "notes.json"
      def get_all_notes():
        # Load all notes from storage
        return storage.load_data(FILENAME)
-It gives the app instant access to the  notes library every time you want to view or search them.
4-the function "add_note" records a new idea or summary, the manager structures it with a title and content:

          def add_note(title, content):
    notes = get_all_notes()
    
    # Assign a unique ID or use length-based numbering
    new_id = notes[-1]["id"] + 1 if notes else 1
    
    new_note = {
        "id": new_id,
        "title": title,
        "content": content
    }
        notes.append(new_note)
    storage.save_data(FILENAME, notes)
    print(f"Note '{title}' saved successfully!")
 -   It keeps your notes organized with clear titles and body text, making them easy to find later.

5-You can view all notes in a clean list or search through them for specific topics:
def view_notes():
    notes = get_all_notes()
   
    if not notes:
        print("Your notes directory is currently empty.")
        return
        
    print("\n--- Study Notes & Summaries ---")
    for note in notes:
        print(f"[{note['id']}] {note['title']}")
        print(f"    {note['content']}")
        print("-" * 30)

   -  It presents the  notes nicely in the terminal so you can review the summaries quickly without opening external apps.

*The "automation.py' file:
  1-While task_manager.py, schedule_manager.py, and notes_manager.py each mind their own business, automation.py looks across all of them at the same time. Its job is to run automated system checks, aggregate your data, and generate a unified status report of your entire study life with a single command.

  2-It imports all the managers and functions from other modules:
           
            import task_manager
            import schedule_manager
           import notes_manager
        
  -It doesn't duplicate code; instead, it coordinates the existing task, schedule, and notes managers to pull data dynamically.

3- the function "generate_system_report" gathers statistics from all three files to show you a bird's-eye view of your progress:
      
      def generate_system_report():
    tasks = task_manager.get_all_tasks()
    schedule = schedule_manager.get_schedule()
    notes = notes_manager.get_all_notes()
    
    # Calculate some quick stats
    total_tasks = len(tasks)
    completed_tasks = sum(1 for t in tasks if t.get("completed", False))
    pending_tasks = total_tasks - completed_tasks
    
    print("\n" + "="*40)
    print("  STUDY ORGANIZER - SYSTEM REPORT ")
    print("="*40)
    print(f"• Total Tasks: {total_tasks} ({completed_tasks} completed, {pending_tasks} pending)")
    print(f"• Scheduled Study Blocks: {len(schedule)}")
    print(f"• Total Notes Saved: {len(notes)}")
    print("="*40)

-Instead of making you check your tasks, schedule, and notes separately, this gives you a fast, automated snapshot of the academic workload in seconds.

4- Automated Health or consistency checks, often, automation modules in CLI tools also check if the system environment is healthy:

              import storage

            def check_system_health():
            # Verify all allowed storage files exist and are readable
            print("Checking storage system health...")
            for filename in storage.ALLOWED_FILES:
             data = storage.load_data(filename)
             print(f" [OK] {filename} loaded successfully ({len(data)} records                 found).")

  - It acts like a diagnostic tool, confirming that the  JSON files are intact and that the app is ready to run without errors.

*The "main.py" file:
  1-main.py is the entry point that ties everything together, displays the interactive menu in the terminal, listens to what you want to do, and routes your choices to the correct manager.
  2-It imports all modules, it act as the master controller, main.py imports everything it needs to coordinate:

            import setup
            import task_manager
            import schedule_manager
            import notes_manager
            import automation
 -  It brings the  entire modular architecture together into one executable script.

 3-The core of main.py is a "while" loop that keeps the app running so you can perform multiple actions without restarting it every time(the interactive menu loop[main]):       

             def main():
    # Ensure setup runs first to create folders/files if missing
    setup.initialize_environment()
    
    while True:
        print("\n=== STUDY ORGANIZER CLI ===")
        print("1. Manage Tasks")
        print("2. Manage Schedule")
        print("3. Manage Notes")
        print("4. Run System Report")
        print("5. Exit")
        
        choice = input("Enter your choice (1-5): ").strip()
        
        if choice == "1":
            handle_tasks_menu()
        elif choice == "2":
            handle_schedule_menu()
        elif choice == "3":
            handle_notes_menu()
        elif choice == "4":
            automation.generate_system_report()
        elif choice == "5":
            print("Exiting Study Organizer. Good luck with your studies!")
            break
        else:
            print("Invalid choice. Please enter a number between 1 and 5.")

-The while True loop keeps the program alive until you explicitly choose option 5 to exit. It also automatically calls setup.initialize_environment() the second it starts, guaranteeing the app never crashes on a missing folder.

4-To keep the main screen from getting cluttered, main.py uses sub-menus for specific modules (like tasks):
           
            def handle_tasks_menu():
    print("\n--- Task Manager ---")
    print("1. View All Tasks")
    print("2. Add New Task")
    print("3. Mark Task Complete")
    
    sub_choice = input("Select an option: ").strip()
    if sub_choice == "1":
        task_manager.get_all_tasks() # (and print them)
    elif sub_choice == "2":
        desc = input("Enter task description: ")
        task_manager.add_task(desc)
  -It keeps the  user interface clean, logical, and easy to navigate directly inside the  terminal.

            
-To run or bash the code, type "python main.py" in the terminal, then multiple choices such as, "manage assignments", "manage tasks", "manage schedule" and more , where it expands into more detai in each corresponding choice.
-Youll be presented by 5 choices, each with a dedicated number , thats how youll control it.
-Lets say you chose to manage tasks, you can then add tasks, view, edit, or even delete the task(s), and each time you add one, it gets an ID number so you can easily navigate through your decisions , if you want to delete or edit.


            

            
    

  

    
    
        
