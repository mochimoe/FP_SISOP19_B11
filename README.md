# FP_SISOP19_B11

##### Buatlah program C yang menyerupai crontab menggunakan daemon dan thread. Ada sebuah file crontab.data untuk menyimpan config dari crontab. Setiap ada perubahan file tersebut maka secara otomatis program menjalankan config yang sesuai dengan perubahan tersebut tanpa perlu diberhentikan. Config hanya sebatas * dan 0-9 (tidak perlu /,- dan yang lainnya) #####


- Library yang digunakan
`````
#include <sys/types.h>
#include <sys/stat.h>
#include <stdio.h>
#include <stdlib.h>
#include <fcntl.h>
#include <errno.h>
#include <unistd.h>
#include <syslog.h>
#include <string.h>
#include <pthread.h>
#include <time.h>

`````

- Fungsi untuk membaca waktu dan eksekusi
`````
void* execute(void* ar) {
    struct croninfo *arg = (struct croninfo *)ar;
    time_t rawtime,check;
    struct tm *currtime;
    while(1) {
        time(&rawtime);
        currtime=localtime(&rawtime);
        if(arg->crontime[0]==currtime->tm_min || arg->crontime[0]<0) {
            if(arg->crontime[1]==currtime->tm_hour || arg->crontime[1]<0) {
                if(arg->crontime[2]==currtime->tm_mday || arg->crontime[2]<0) {
                    if(arg->crontime[3]==currtime->tm_mon+1 || arg->crontime[3]<0) {
                        if(arg->crontime[4]==currtime->tm_wday || arg->crontime[4<0]) {
                            // puts(arg->line);
                            puts(arg->path);
                            pid_t child;
                            child=fork();
                            if(child==0) {
                                char *argv[]={"bash",arg->path,NULL};
                                execv("/bin/bash",argv);
                            }
                            else {
                                puts("success");
                            }
                        }
                    }
                }
            }
        }
        time(&check);
        while(difftime(check,rawtime)<=60.0) {
            sleep(1);
            time(&check);
        }
    }
}

`````

- fungsi getconfig
`````
int getconfig(int line,char config[]) {
    int i=0,cpy=0;
    char a[2];

    // puts(".");

    while(i < 5) {
        if(config[cpy]==' ') {
            cpy++;
        }
        else if(config[cpy]!='*') {
            a[0]=config[cpy];
            cron[line].crontime[i]=atoi(a);
            cpy++;
            i++;
        }  
        else {
            cron[line].crontime[i]=-1;
            cpy++;
            i++;
        }
        // puts(".");
    }

    char *path=strrchr(config,' ');
    char *temp=strchr(path,'/');
    strcpy(cron[line].path,temp);
}


`````
- Fungsi getLine
`````
int getLines(FILE* file,int line) {
    char ch;
    int i=0;
    char config[128];
    pthread_t threads[10];
    memset(config,'\0',sizeof(config));

    while(fscanf(file,"%c",&ch)!=EOF) {
        if(ch=='\n') {
            strcpy(cron[line].line,config);
            getconfig(line,config);
            // cron.lines=line;
            // puts(cron[line].line);
            struct croninfo *pcron = &cron[line];
            pthread_create(&threads[line],NULL,execute,(void*)pcron);
            maxline=line;
            line++;
            i=0;
            memset(config,'\0',sizeof(config));
        }
        else {
            config[i]=ch;
            i++;
        }
    }
    // puts(cron.line);
    return 0;
}

`````

- Fungsi main dan daemon

`````
int main() {
  pid_t pid, sid;
  int lastmod=0;
  char line[10];
  FILE* cronfile;

  pid = fork();

  if (pid < 0) {
    exit(EXIT_FAILURE);
  }

  if (pid > 0) {
    exit(EXIT_SUCCESS);
  }

  umask(0);

  sid = setsid();

  if (sid < 0) {
    exit(EXIT_FAILURE);
  }

  if ((chdir("/")) < 0) {
    exit(EXIT_FAILURE);
  }

  close(STDIN_FILENO);
  close(STDOUT_FILENO);
  close(STDERR_FILENO);

    while(1) {
      struct stat st;
        stat(loc,&st);

        if(lastmod!=st.st_mtime) {
            lastmod=st.st_mtime;
            
            int count=0;
            cronfile = fopen(loc,"r");

            getLines(cronfile,0);


            // puts(config);
            // execute();
        }
    }
  exit(EXIT_SUCCESS);
}

`````
