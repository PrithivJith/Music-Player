#module imports
import os,pygame,keyboard,time,threading
import tkinter as tk
from tkinter import filedialog
from PIL import ImageTk, Image
from pygame import mixer

#module initialise
mixer.init()
pygame.init()

#music file path and initial directory path and song file name
path = ""
initial_folder_path = ""
filename= ""
#default and current keybinds dictionary
keybinds = {"pause":"","stop":"","vol_up":"","vol_down":""}
default_keybinds = {"pause":"home","stop":"end","vol_up":"up","vol_down":"down"}
#vaild keys on keyboard list
keys = [
        # Letter keys
        'A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'K', 'L', 'M',
        'N', 'O', 'P', 'Q', 'R', 'S', 'T', 'U', 'V', 'W', 'X', 'Y', 'Z',

        # Number keys
        '0', '1', '2', '3', '4', '5', '6', '7', '8', '9',

        # Function keys
        'F1', 'F2', 'F3', 'F4', 'F5', 'F6', 'F7', 'F8', 'F9', 'F10', 'F11', 'F12',

        # Modifier keys
        'CTRL', 'SHIFT', 'ALT', 'META',

        # Arrow keys
        'LEFT', 'UP', 'RIGHT', 'DOWN',

        # Special keys
        'ESCAPE', 'TAB', 'ENTER', 'BACKSPACE', 'SPACE', 'CAPSLOCK', 'DELETE',
        'INSERT', 'HOME', 'END', 'PAGEUP', 'PAGEDOWN',

        # Symbol keys
        '`', '-', '=', '[', ']', ';', "'", ',', '.', '/',

        # Numpad keys
        'NUMLOCK', 'KP0', 'KP1', 'KP2', 'KP3', 'KP4', 'KP5', 'KP6', 'KP7', 'KP8', 'KP9',
        'KP_DIVIDE', 'KP_MULTIPLY', 'KP_SUBTRACT', 'KP_ADD', 'KP_DECIMAL', 'KP_ENTER'
    ]
#content of keybinds.txt file
keybinds_content = []

#saves current keybind dictionary to a txt file.(use new lines to represent keys;{line0:"pause",line1"stop",line2:"voulum_up")
def save():
    f = open("keybinds.txt", "w")
    f.write(keybinds["pause"]+"\n")
    f.write(keybinds["stop"]+"\n")
    f.write(keybinds["vol_up"]+"\n")
    f.write(keybinds["vol_down"] + "\n")
    f.close()

#sets current keybinds dict to default keybinds dict.
def reset_keys():
    keybinds = default_keybinds
    for item in all_entry:
        set_entry_text(keybinds[item.winfo_name()], item)
#checks if keybind is in keys list.
def check_for_vaild_keybind(content_to_check):
    if content_to_check.upper() in keys:
        return True
    else:
        return False

#reads keybinds.txt and sets it to current keybinds
def set_keybinds_file():
    global keybinds_content
    global keybinds
    #stores each line of file in a list
    f = open("keybinds.txt")
    keybinds_content = f.readlines()
    #enumerates through this list with index and item
    for line_number,line in enumerate(keybinds_content):
        #each line number represents keys and is passed through keybinds dict
        match line_number:
            case 0:
                if check_for_vaild_keybind(line.replace('\n','')):
                    keybinds["pause"] = line.replace('\n','')
            case 1:

                if check_for_vaild_keybind(line.replace('\n','')):
                    keybinds["stop"] = line.replace('\n','')
            case 2:
                if check_for_vaild_keybind(line.replace('\n','')):
                    keybinds["vol_up"] = line.replace('\n','')
            case 3:
                if check_for_vaild_keybind(line.replace('\n', '')):
                    keybinds["vol_down"] = line.replace('\n', '')
        #sets entry text on visual side.
        set_entry_text(None,"all")
    f.close()
    return keybinds_content

def set_entry_text(txt,obj):
    #if object is a widget then txt arg will be inserted as its text.
    if isinstance(obj,tk.Entry):
        obj.delete(0, tk.END)
        obj.insert(0,txt)
    #if object is a string and is "all" then sets the text to its current keybind.
    elif isinstance(obj,str) and obj == "all":
        for item in all_entry:
            set_entry_text(keybinds[item.winfo_name()], item)

#returns how many windows are children to the main window.
def num_of_windows(root:tk.Tk):
    children = root.winfo_children()
    return sum(1 for child in children if isinstance(child,tk.Toplevel))

#shows help window
def help_prompt():
    # if the window is not aldready open.
    if num_of_windows(window) == 0:

        #set up help window
        BACKGROUND_COLOR_HELP = "#87B1E8"
        prompt_window = tk.Toplevel()
        prompt_window.title("Help")
        prompt_window.geometry("620x350")
        prompt_window.resizable(False, False)
        prompt_window.config(background=BACKGROUND_COLOR_HELP)

        #set up help widgets
        help_label = tk.Label(prompt_window,font=("Cascadia Code SemiLight", 16,"underline bold"),text="Help",bg=BACKGROUND_COLOR_HELP)
        keybinds_help = tk.Label(prompt_window,font=("Cascadia Code SemiLight", 13,"underline"),text="Keybinds",bg=BACKGROUND_COLOR_HELP)
        keybinds_help_bull1 = tk.Label(prompt_window,font=("Cascadia Code SemiLight", 11),bg=BACKGROUND_COLOR_HELP,text="-Click on each entry field to enter a shortcut.")
        keybinds_help_bull2 = tk.Label(prompt_window, font=("Cascadia Code SemiLight", 11), bg=BACKGROUND_COLOR_HELP,text="-Press the 'esc' key to remove focus from the entry fields.")
        keybinds_help_bull3 = tk.Label(prompt_window, font=("Cascadia Code SemiLight", 11), bg=BACKGROUND_COLOR_HELP,text="-Press the 'save' button to save your keybinds.")
        keybinds_help_bull4 = tk.Label(prompt_window, font=("Cascadia Code SemiLight", 11), bg=BACKGROUND_COLOR_HELP,text="-Press the 'reset' button to reset your keybinds to default.")

        control_label = tk.Label(prompt_window,font=("Cascadia Code SemiLight", 13,"underline"),text="Controls",bg=BACKGROUND_COLOR_HELP)
        control_label_bull1 = tk.Label(prompt_window,font=("Cascadia Code SemiLight", 11),bg=BACKGROUND_COLOR_HELP,text="-Click on 'loop checkbox' to enable song looping.")
        control_label_bull2 = tk.Label(prompt_window, font=("Cascadia Code SemiLight", 11), bg=BACKGROUND_COLOR_HELP,text="-Press the 'play' button to play your song.")

        file_label = tk.Label(prompt_window,font=("Cascadia Code SemiLight", 13,"underline"),text="Files",bg=BACKGROUND_COLOR_HELP)
        file_label_bull1 = tk.Label(prompt_window, font=("Cascadia Code SemiLight", 11), bg=BACKGROUND_COLOR_HELP,text="-Press the 'folder' icon to select your song file.")
        file_label_bull2 = tk.Label(prompt_window, font=("Cascadia Code SemiLight", 11), bg=BACKGROUND_COLOR_HELP,text="-Press the 'Default folder' button to set where your song files are.")


        #place help widgets
        help_label.place(x=279,y=5)
        keybinds_help.place(x=5,y=50)
        keybinds_help.lower()
        keybinds_help_bull1.place(x=5,y=80)
        keybinds_help_bull1.lower()
        keybinds_help_bull2.place(x=5, y=105)
        keybinds_help_bull3.place(x=5,y=130)
        keybinds_help_bull4.place(x=5,y=155)

        control_label.lower()
        control_label.place(x=5,y=185)
        control_label_bull1.place(x=5,y=215)
        control_label_bull2.place(x=5,y=240)

        file_label.place(x=5,y=270)
        file_label_bull1.place(x=5,y=295)
        file_label_bull2.place(x=5,y=320)

#gets entry in a while loop.
def get_entry(entry):
    while True:
        if check_for_vaild_keybind(entry.get()):
            entry.config(fg="black")
            keybinds[entry.winfo_name()] = entry.get()
        else:
            entry.config(fg="red2")
        time.sleep(0.007)

#if the esc key is pressed the entry will not be focuesed.
def focus_entry(entry):
    while True:
        if keyboard.is_pressed('esc'):
            time.sleep(0.1)
            set_entry_text(keybinds[entry.winfo_name()],entry)
            window.focus()
        time.sleep(0.01)

supported_formats = (".mp3", ".wav", ".ogg")
f = open("root_path.txt","r")
content = f.readlines()
#tries to read initial folder path if the txt doc is not blank
try:
    if content[0] != '':
        initial_folder_path = content[0]
        print(initial_folder_path)
except IndexError:
    print("Text document is empty")
f.close()

#opens the song file prompt
def openFile():
    global path
    global initial_folder_path
    print(initial_folder_path)
    path = ""
    global filename
    path = filedialog.askopenfilename(initialdir=initial_folder_path)
    print(path)
    if path.lower().endswith(supported_formats):
        file_name = os.path.basename(path)
        song_path_text.config(text=file_name, fg="black")
    else:
        path = ""
        song_path_text.config(text="Invalid Format", fg="red2")
#opens the initial directory prompt
def root_directory():
    global initial_folder_path
    initial_folder_path = filedialog.askdirectory()
    f = open("root_path.txt","w")
    f.write(initial_folder_path)

playing = False
#creates a func that plays the music in a different thread.
def play_music():
    # makes list of all dict values and checks if all is true(if there is not an empty string).
    if all([n for n in keybinds.values()]):
        threading.Thread(target=play,daemon=True).start()

def play():
    global playing
    if path != "":
        def wait_for_key_release(key):
            while keyboard.is_pressed(key):
                pass
        #default vars for pause and vol
        paused = False
        volume = 0.5

        #loads and starts music
        mixer.music.load(path)
        mixer.music.set_volume(volume)
        mixer.music.play()


        playing = True
        while True:
            #makes a custom event call when music ends
            mixer.music.set_endevent(pygame.USEREVENT)
            for event in pygame.event.get():
                if event.type == pygame.USEREVENT and loop.get()==1:
                    # Restart the song when it finishes playing
                    pygame.mixer.music.rewind()
                    pygame.mixer.music.play()
                else:
                    #stops song
                    mixer.music.stop()
                    break

            if keyboard.is_pressed(keybinds["pause"]):
                if paused:
                    #resumes
                    mixer.music.unpause()
                    paused = False
                    print("Resumed")
                else:
                    #pauses
                    mixer.music.pause()
                    paused = True
                    print("Paused")
                wait_for_key_release(keybinds["pause"])

            if keyboard.is_pressed(keybinds["vol_up"]):
                #vol up till limit
                volume += 0.035
                if volume > 3:
                    volume = 1
                mixer.music.set_volume(volume)
                print("Volume increased to", volume)
                time.sleep(0.1)
            if keyboard.is_pressed('down'):
                # vol down till limit
                volume -= 0.025
                if volume < 0:
                    volume = 0
                mixer.music.set_volume(volume)
                print("Volume decreased to", volume)
                time.sleep(0.1)
            if keyboard.is_pressed(keybinds["stop"]):
                #stops music
                mixer.music.stop()
                break
            time.sleep(0.01)
            playing =True
        playing = False

#bg color constant
BACKGROUND_COLOR = "#86c3d9"

#sets up window
window = tk.Tk()
window.geometry("600x350")
window.title("Song player -By Prithiv")
window.resizable(False, False)
window.config(background=BACKGROUND_COLOR)

#sets up images
og_file_explorer_image = Image.open("file.png")
resized = og_file_explorer_image.resize((23, 23))
file_explorer_image = ImageTk.PhotoImage(resized)

og_root_image = Image.open("root_dir.png")
resized_root = og_root_image.resize((103,27))
root_image = ImageTk.PhotoImage(resized_root)

og_reset_image = Image.open("reset.png")
resized_reset = og_reset_image.resize((68,30))
reset_image = ImageTk.PhotoImage(resized_reset)

og_save_image = Image.open("save.png")
resized_save = og_save_image.resize((64,30))
save_image = ImageTk.PhotoImage(resized_save)

og_play_image = Image.open("play.png")
resized_play = og_play_image.resize((93,37))
play_image = ImageTk.PhotoImage(resized_play)

og_help_image = Image.open("help.jpg")
resized_help = og_help_image.resize((30,30))
help_image = ImageTk.PhotoImage(resized_help)

#sets up widgets

file_explorer_button = tk.Button(window,image=file_explorer_image, command=openFile, borderwidth=0, bg=BACKGROUND_COLOR, activebackground=BACKGROUND_COLOR)

play_button = tk.Button(window,command=play_music, image=play_image,borderwidth=0,bg=BACKGROUND_COLOR,activebackground=BACKGROUND_COLOR)

save_button = tk.Button(window,borderwidth=0,bg=BACKGROUND_COLOR,activebackground=BACKGROUND_COLOR,image=save_image,command=save)

root_directory_button = tk.Button(window,command=root_directory,image=root_image,borderwidth=0,bg=BACKGROUND_COLOR,activebackground=BACKGROUND_COLOR)

loop = tk.IntVar()
loop_checkbox = tk.Checkbutton(window,bg=BACKGROUND_COLOR,activebackground=BACKGROUND_COLOR,variable=loop,onvalue=1,offvalue=0,height=1,width=1)
loop_label = tk.Label(window,text="Loop:",font=("Cascadia Code SemiLight", 13),bg=BACKGROUND_COLOR)

reset_keybinds = tk.Button(window,image=reset_image, borderwidth=0,bg=BACKGROUND_COLOR,activebackground=BACKGROUND_COLOR,command=reset_keys)

pause = tk.Entry(window,width=10,name="pause")
pause_label = tk.Label(window,font=("Cascadia Code SemiLight", 13),text="Pause:",bg=BACKGROUND_COLOR)

stop = tk.Entry(window,width=10,name="stop")
stop_label = tk.Label(window,font=("Cascadia Code SemiLight", 13),text="Stop:",bg=BACKGROUND_COLOR)

song_path_text = tk.Label(window,width=50, text="")
path_label = tk.Label(window,bg=BACKGROUND_COLOR, fg="black", text="Song name: ", font=("Cascadia Code SemiLight", 12, "bold"))

vol_up = tk.Entry(window,width=10,name="vol_up")
vol_up_label = tk.Label(window,font=("Cascadia Code SemiLight", 11),text="Volume up:",bg=BACKGROUND_COLOR)

vol_down = tk.Entry(window,width=10,name="vol_down")
vol_down_label = tk.Label(window,font=("Cascadia Code SemiLight", 11),text="Volume down:",bg=BACKGROUND_COLOR)

keybinds_label = tk.Label(window,font=("Cascadia Code SemiLight", 16,"underline"),text="Keybinds",bg=BACKGROUND_COLOR)

help = tk.Button(window,image=help_image, borderwidth=0,bg=BACKGROUND_COLOR,activebackground=BACKGROUND_COLOR,command=help_prompt)

#places widgets

vol_down_label.place(x=225,y=265)
vol_down.place(x=340,y=270)

vol_up_label.place(x=225,y=212)
vol_up.place(x=330,y=217)

stop_label.place(x=86,y=265)
stop.place(x=142,y=270)

pause.place(x=150,y=217)
pause_label.place(x=85,y=212)

song_path_text.place(x=105, y=20)
path_label.place(x=1, y=15)
file_explorer_button.place(x=463, y=18)
root_directory_button.place(x=490,y=319)

loop_label.place(x=10,y=80)
loop_checkbox.place(x=60,y=85)

reset_keybinds.place(x=177,y=315)
save_button.place(x=257,y=315)

play_button.place(x=255, y=50)

keybinds_label.place(x=195,y=160)

help.place(x=5,y=315)
#starts focus entry and get entry in different threads
threading.Thread(target=focus_entry,args=(stop,),daemon=True).start()
threading.Thread(target=focus_entry,args=(vol_up,),daemon=True).start()
threading.Thread(target=focus_entry,args=(pause,),daemon=True).start()
threading.Thread(target=focus_entry,args=(vol_down,),daemon=True).start()

threading.Thread(target=get_entry,args=(vol_down,),daemon=True).start()
threading.Thread(target=get_entry,args=(pause,),daemon=True).start()
threading.Thread(target=get_entry,args=(vol_up,),daemon=True).start()
threading.Thread(target=get_entry,args=(stop,),daemon=True).start()


all_entry = [pause,stop,vol_up,vol_down]

# reads keybinds file when program starts
set_keybinds_file()

window.mainloop()
