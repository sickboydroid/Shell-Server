#!/bin/python3
import sys, os, shutil, pathlib, subprocess as proc
from os import path
from subprocess import run, PIPE
from tempfile import mktemp
from threading import Thread

# constants
SERVER_DIR = path.join(str(pathlib.Path.home()), '.server')
BACKUP_DIR = path.join(SERVER_DIR, '.backup')

UPLOAD_REPO_URL = 'git@github.com:game-of-coding/termux-server.git'
DOWNLOAD_REPO_URL = 'git@github.com:game-of-coding/cloudshell-server.git'
# UPLOAD_REPO_URL = 'git@github.com:game-of-coding/cloudshell-server.git'
# DOWNLOAD_REPO_URL = 'git@github.com:game-of-coding/termux-server.git'

UPLOAD_REPO_DIR = path.join(SERVER_DIR, 'upload_server')
DOWNLOAD_REPO_DIR = path.join(SERVER_DIR, 'download_server')

class Colors:
    OKBLUE = '\033[94m'
    OKGREEN = '\033[92m'
    WARNING = '\033[93m'
    FAIL = '\033[91m'
    BOLD = '\033[1m'
    UNDERLINE = '\033[4m'
    ENDC = '\033[0m'

# print utils
class PrintUtils:
    @staticmethod
    def printRed(msg, end='\n'):
        PrintUtils.printWithColor(Colors.FAIL, msg, end)

    @staticmethod
    def printGreen(msg, end='\n'):
        PrintUtils.printWithColor(Colors.OKGREEN, msg, end)
 
    @staticmethod
    def printBlue(msg, end='\n'):
        PrintUtils.printWithColor(Colors.OKBLUE, msg, end)
        
    @staticmethod
    def printBold(msg, end='\n'):
        PrintUtils.printWithColor(Colors.BOLD, msg, end)

    @staticmethod
    def printWithColor(color, msg, end):
        print(f"{color}{msg}{Colors.ENDC}", end=end)

# Command line utils
class CmdLine:
    def __init__(self):
        self.returncode = 0

    def cmd(self, command):
        # Temporery files for getting output
        stdoutfile = open(mktemp(), 'w')
        stderrfile = open(mktemp(), 'w')

        # Run command in background
        thread = Thread(target=self.cmd_in_background, args=(command, stdoutfile, stderrfile,))
        thread.start()

        # Read and print output to screen continously (like tail -f file_name)
        hadoutput = False
        with open(stdoutfile.name) as stdoutfile, open(stderrfile.name) as stderrfile:
            stdoutfile.seek(0,2)
            stderrfile.seek(0,2)
            while thread.is_alive():
                # stdout
                outline = stdoutfile.readline()
                if outline:
                    if not hadoutput:
                        hadoutput = True
                    print(outline, end='')
                # stderr
                errline = stderrfile.readline()
                if errline:
                    if not hadoutput:
                        hadoutput = True
                    print(errline, end='')
        stdoutfile.close()
        stderrfile.close()
        return self.CmdOutput(self.returncode, hadoutput)
    
    def cmd_in_background(self, command, stdoutfile, stderrfile):
        self.returncode = proc.run(command, shell=True,
                              stdout=stdoutfile, stderr=stderrfile,
                              universal_newlines=True).returncode

    class CmdOutput:
        def __init__(self, returncode, hadoutput):
            self.returncode = returncode
            self.hadoutput = hadoutput

def download_files():
    command = f'cd {DOWNLOAD_REPO_DIR} && git pull && cd - > /dev/null && cp -r {DOWNLOAD_REPO_DIR}/* .'
    output = CmdLine().cmd(command)
    if output.returncode == 0:
        print('Copied to current folder')
    else:
        PrintUtils.printRed('Failed to download files')

def upload_files():
    # Get files to upload
    args = sys.argv[2:]
    files = None
    if len(args) <= 0:
        files = os.listdir()
        if len(files) <= 0:
            PrintUtils.printRed('Error: Current directory is empty, no files to upload')
            return
    else:
        files = args
        all_files_exist = True
        for item in files:
            if not path.exists(item):
                PrintUtils.printRed(f'Error: File \'{item}\' does not exist')
                all_files_exist = False
        if not all_files_exist:
            print('Fix above errors to continue')
            return

    # Build command to copy all files to server directory
    command = f'mv {UPLOAD_REPO_DIR}/* {BACKUP_DIR} &&  cp -r'
    for item in files:
        command += ' ' + item
    command += f' {UPLOAD_REPO_DIR}'

    # Run copy command
    output = CmdLine().cmd(command)
    if output.returncode != 0:
        PrintUtils.printRed('Failed to copy files for upload')
        print('command = ' + command)
        return

    # Upload files to server
    command = f'cd {UPLOAD_REPO_DIR} && git add -A && git commit -m \'server\' && git push && cd - > /dev/null'
    output = CmdLine().cmd(command)
    if output.returncode != 0:
        PrintUtils.printRed('Failed to upload files')
        return

def print_help():
    print('usage: server [h] [d] [u] <args>')
    print('\th \t\t: Show this help')
    print('\td <directory>\t: Download uploaded files to the mentioned directory or to the current directory')
    print('\tu <file...>\t: Upload passed file(s) or current directory')
    
def is_git_repo(direc):
       if not path.isdir(direc):
           return False
       if not path.exists(path.join(direc, '.git')):
           return False
       return True

def clone_repo(repo_url, dest_dir):
    if path.exists(dest_dir):
        if len(os.listdir(dest_dir)) > 0:
            print('Cleaning repo. directory...')
            shutil.rmtree(dest_dir)
            os.mkdir(dest_dir)
            print('Done')
    print('Cloning server directory...')
    output = CmdLine().cmd(f'git clone {repo_url} {dest_dir} && cd {dest_dir} && git config pull.rebase true && cd - > /dev/null')
    if output.returncode != 0:
        return False
    print('Done')
    return True

def do_initial_setup():
    if not path.exists(SERVER_DIR):
        os.mkdir(SERVER_DIR)
    if not path.exists(BACKUP_DIR):
        os.mkdir(BACKUP_DIR)

def reset_server():
    choice = str(input('Do you really want to reset upload server? [Y/n]: ')).lower()
    if choice == 'y' or choice == 'yes':
        command = 'rm -rf {UPLOAD_REPO_DIR} {DOWNLOAD_REPO_DIR}'
        output = CmdLine().cmd(command)
        if output.returncode != 0:
            PrintUtils.printRed('Error: Failed to delete server directories')
        else:
            PrintUtils.printGreen('Successfully deleted directories...')
            PrintUtils.printBold('NOTE: Please delete \'TERMUX-SERVER\' and \'CLOUDSHELL-SERVER\' repos on github and then recreate them...')
    else:
        print('Aborted!')
   
def clone_repos():
    has_cloned_any_repo = False
    if not is_git_repo(UPLOAD_REPO_DIR):
        print('*'*30)
        if not clone_repo(UPLOAD_REPO_URL, UPLOAD_REPO_DIR):
            print('*'*30)
            return False
        else:
            has_cloned_any_repo = True
    if not is_git_repo(DOWNLOAD_REPO_DIR):
        print('*'*30)
        if not clone_repo(DOWNLOAD_REPO_URL, DOWNLOAD_REPO_DIR):
            print('*'*30)
            return False
        else:
            has_cloned_any_repo = True
    if has_cloned_any_repo:
        print('*'*30)
    return True

def main():
    do_initial_setup()
    if not clone_repos():
        return

    # Manipulate passed arguments
    args = sys.argv[1:]
    if len(args) <= 0:
        PrintUtils.printRed('NOTHING TO DO')
        print('*'*30)
        print_help()
        print('*'*30)
        return

    if args[0] == 'd':
        print('*'*10, 'Download', '*'*10)
        download_files()
    elif args[0] == 'u':
        print('*'*11, 'Upload', '*'*11)
        upload_files()
    elif args[0] == 'reset':
        print('*'*8, 'Reset Server', '*'*8)
        reset_server()
    elif args[0] == 'h':
        print('*'*12, 'Help', '*'*12)
        print_help()
    else:
        print('*'*8, 'Unknown task', '*'*8)
        PrintUtils.printRed(f'Error: Unknow task \'{args[0]}\'')
        print('Use \'server h\' for reading help page')
    print('*'*30)

if __name__ == '__main__':
    main()
