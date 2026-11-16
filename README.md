#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>

int main(void)
{
    int pipe1[2], pipe2[2];
    pid_t pid;
    char buffer[100];

    char parentMessage[] = "How are you? What's your VUID?";
    char childMessage[] = "I am fine. My VUID is BC240412107";

    if (pipe(pipe1) == -1 || pipe(pipe2) == -1)
    {
        perror("Pipe creation failed");
        return 1;
    }

    pid = fork();

    if (pid < 0)
    {
        perror("Fork failed");
        return 1;
    }

    if (pid == 0)
    {
        // Child process
        close(pipe1[1]);
        close(pipe2[0]);

        memset(buffer, 0, sizeof(buffer));

        if (read(pipe1[0], buffer, sizeof(buffer)) < 0)
        {
            perror("Child read failed");
            exit(1);
        }

        printf("Child received: %s\n\n", buffer);
        fflush(stdout);

        if (write(pipe2[1], childMessage,
                  strlen(childMessage) + 1) == -1)
        {
            perror("Child write failed");
            exit(1);
        }

        close(pipe1[0]);
        close(pipe2[1]);

        exit(0);
    }
    else
    {
        // Parent process
        close(pipe1[0]);
        close(pipe2[1]);

        if (write(pipe1[1], parentMessage,
                  strlen(parentMessage) + 1) == -1)
        {
            perror("Parent write failed");
            return 1;
        }

        close(pipe1[1]);

        memset(buffer, 0, sizeof(buffer));

        if (read(pipe2[0], buffer, sizeof(buffer)) < 0)
        {
            perror("Parent read failed");
            return 1;
        }

        printf("Parent received: %s\n", buffer);

        close(pipe2[0]);
        wait(NULL);
    }

    return 0;
}
