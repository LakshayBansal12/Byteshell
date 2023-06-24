#include <iostream>
#include <vector>
#include <string>
#include <cstring>
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <memory>
using namespace std;

/*
  Function Declarations for builtin shell commands:
 */

int shell_change_directory(vector<string>& args); // Renamed to "shell_change_directory"
int shell_help(vector<string>& args);
int shell_exit(vector<string>& args);
int shell_history(vector<string>& args);

/*
  List of builtin commands, followed by their corresponding functions.
 */

vector<string> builtin_str = {
    "cd",
    "help",
    "exit",
    "history"
};

int (*builtin_func[])(vector<string>&) = {
    &shell_change_directory,
    &shell_help,
    &shell_exit,
    &shell_history
};

struct Node
{
    string str;
    shared_ptr<Node> next;
};

shared_ptr<Node> head = nullptr;
shared_ptr<Node> cur = nullptr;

// All function present in the below code.
void add_to_hist(vector<string>& args);
string strAppend(const string& str1, const string& str2);
int shell_history(vector<string>& args);
int shell_num_builtins();
int shell_change_directory(vector<string>& args); // Renamed to "shell_change_directory"
int shell_help(vector<string>& args);
int shell_exit(vector<string>& args);
int shell_launch(vector<string>& args);
int shell_execute(vector<string>& args);
string shell_read_line();
vector<string> shell_split_line(const string& line);

void add_to_hist(vector<string>& args)
{
    shared_ptr<Node> ptr = make_shared<Node>();

    if (head == nullptr)
    {
        head = make_shared<Node>();
        head->str = "";

        string str1 = " ";

        if (!args.empty())
            head->str = strAppend(args[0], str1);

        if (args.size() > 1)
            head->str = strAppend(head->str, args[1]);

        head->next = nullptr;
        cur = head;
    }
    else
    {
        ptr = make_shared<Node>();
        string str1 = " ";

        if (!args.empty())
            ptr->str = strAppend(args[0], str1);

        if (args.size() > 1)
            ptr->str = strAppend(ptr->str, args[1]);

        cur->next = ptr;
        ptr->next = nullptr;
        cur = ptr;
    }
}

string strAppend(const string& str1, const string& str2)
{
    return str1 + str2;
}

int shell_history(vector<string>& args)
{
    shared_ptr<Node> ptr = head;
    int i = 1;
    while (ptr != nullptr)
    {
        cout << " " << i++ << " " << ptr->str << endl;
        ptr = ptr->next;
    }
    return 1;
}

int shell_num_builtins()
{
    return builtin_str.size();
}

int shell_change_directory(vector<string>& args)
{
    if (args.size() < 2)
    {
        cerr << "lsh: expected argument to \"cd\"" << endl;
    }
    else
    {
        if (chdir(args[1].c_str()) != 0)
        {
            perror("lsh");
        }
    }
    return 1;
}

int shell_help(vector<string>& args)
{
    cout << "Type program names and arguments, and hit enter." << endl;
    cout << "The following are built-in commands:" << endl;

    for (const auto& command : builtin_str)
    {
        cout << "  " << command << endl;
    }

    cout << "Use the man command for information on other programs." << endl;
    return 1;
}

int shell_exit(vector<string>& args)
{
    return 0;
}

int shell_launch(vector<string>& args)
{
    pid_t pid;
    int status;

    pid = fork();
    if (pid == 0)
    {
        // Child process
        vector<char*> c_args(args.size() + 1);
        for (size_t i = 0; i < args.size(); ++i)
        {
            c_args[i] = const_cast<char*>(args[i].c_str());
        }
        c_args[args.size()] = nullptr;

        if (execvp(c_args[0], c_args.data()) == -1)
        {
            perror("lsh");
            exit(EXIT_FAILURE); // Add exit here to terminate child process on execvp error
        }
        exit(EXIT_FAILURE); // Move this exit outside the if block
    }
    else if (pid < 0)
    {
        // Error forking
        perror("lsh");
    }
    else
    {
        // Parent process
        do
        {
            waitpid(pid, &status, WUNTRACED);
        } while (!WIFEXITED(status) && !WIFSIGNALED(status));
    }

    return 1;
}

// Function that takes a list of arguments and executes the command
int shell_execute(vector<string>& args)
{
    if (args.empty())
    {
        // An empty command was entered.
        return 1;
    }

    for (size_t i = 0; i < shell_num_builtins(); i++)
    {
        if (args[0] == builtin_str[i])
        {
            return (*builtin_func[i])(args);
        }
    }

    return shell_launch(args);
}

// Function to take command from user input.
string shell_read_line()
{
    string command;
    if (!getline(cin, command))
    {
        if (cin.eof())
        {
            exit(EXIT_SUCCESS); // Received an EOF. Hence go out of loop
        }
        else
        {
            perror("lsh: getline\n");
            exit(EXIT_FAILURE);
        }
    }
    return command;
}

// Function to change the user command into a list(vector) of arguments.
#define SHELL_TOK_BUFSIZE 64
#define SHELL_TOK_DELIM " \t\r\n\a"

vector<string> shell_split_line(const string& line)
{
    vector<string> tokens;
    size_t index = 0;
    size_t found = 0;
    while ((found = line.find_first_of(SHELL_TOK_DELIM, index)) != string::npos)
    {
        if (found != index)
        {
            tokens.push_back(line.substr(index, found - index));
        }
        index = found + 1;
    }
    if (index < line.size())
    {
        tokens.push_back(line.substr(index));
    }
    return tokens;
}

void shell_loop()
{
    string command;
    vector<string> command_tokenize;
    int status = 1;

    do
    {
        cout << "> ";
        command = shell_read_line();
        command_tokenize = shell_split_line(command);
        add_to_hist(command_tokenize);
        status = shell_execute(command_tokenize);
    } while (status);
}

int main(int argc, char** argv)
{
    // Run command loop.
    shell_loop();

    return EXIT_SUCCESS;
}
