#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <arpa/inet.h>
#include <unistd.h>
#include <signal.h>
#include <syslog.h>
#include <fcntl.h>
#include <sys/select.h>

#define PORT_TCP 8000	//Komunikacja przez TCP
#define PORT2 14	//Odbieranie danych UDP
#define PORT 12345	//Wysylanie danych UDP
#define BUFFER_SIZE 2000
#define PASSWORD "123"		//Serwer przechowuje hasla, rowniez mozna zmienic
#define MAXFD 64

//Obsluga demona, kod z zajec "Programowanie sieciowe"
int daemon_init(){
	int i,p;
	pid_t pid;

	if( (pid = fork()) < 0){
		return -1;
	}

	else if(pid){
		exit(0);
	}

	if( setsid() < 0){
		return -1;
	}

	signal(SIGHUP, SIG_IGN);
	if ((pid = fork() < 0)){
		return -1;
	}

	else if (pid){
		exit(0);
	}

	chdir("/tmp");

	p = open("/dev/null", O_RDONLY);
	open("/dev/null", O_RDWR);
	open("/dev/null", O_RDWR);

	return (0);
}


//Zapisz do pliku
void save(const char *message, char* filename){
	FILE *file = fopen(filename, "a");
	if(file == NULL){
		perror("Blad otwarcia");
		return;
	}
	fprintf(file, "%s\n", message);
	fclose(file);
}

//Wczytaj z pliku
void load(char *buffer, size_t buffer_size, char* filename){
	FILE *file = fopen(filename, "r");
	if(file == NULL){
		sprintf(buffer,"Brak histori.");
		return;
	}

	fread(buffer, 1, buffer_size - 1, file);
	buffer[buffer_size - 1] = '\0';
	fclose(file);
}





int main(int argc, char **argv) {
	int flaga = 0;  //Ustawiona w zaleznosci od wyboru pokoju przez klienta
	fd_set read_fds;
	int ip, max_fd;
	int send_fd, recv_fd, tcp_fd, client_fd;
	int n = 20;
	char buffer[BUFFER_SIZE], buffer_tcp[BUFFER_SIZE], buffer2[BUFFER_SIZE];
	struct sockaddr_in group_addr;
	char password[BUFFER_SIZE] = PASSWORD;
	struct sockaddr_in server_addr, client_addr, tcp_addr;
	socklen_t addr_len = sizeof(group_addr);
	char wybor[BUFFER_SIZE];
	char file_name[BUFFER_SIZE] = "Plik";

    //    daemon_init();	//Odkomentowac, aby wlaczyc demona


	recv_fd = socket(AF_INET, SOCK_DGRAM, 0);
	send_fd = socket(AF_INET, SOCK_DGRAM, 0);
	tcp_fd = socket(AF_INET, SOCK_STREAM, 0); 

	//Ustawienie adresow, sin_addr.s_addr zostanie przypipsany po odebraniu zadania kanalu od klienta
	memset(&group_addr, 0, sizeof(group_addr));
	group_addr.sin_family = AF_INET;
	group_addr.sin_port = htons(PORT);


	memset(&server_addr, 0, sizeof(server_addr));
	server_addr.sin_family = AF_INET;
	server_addr.sin_addr.s_addr = INADDR_ANY;
	server_addr.sin_port = htons(PORT2);

	//Adres do komunikacji przez TCP
        memset(&tcp_addr, 0, sizeof(tcp_addr));
        tcp_addr.sin_family = AF_INET;
        tcp_addr.sin_addr.s_addr = INADDR_ANY;
        tcp_addr.sin_port = htons(PORT_TCP);


	bind(tcp_fd, (const struct sockaddr *)&tcp_addr, sizeof(tcp_addr));
	bind(recv_fd, (const struct sockaddr *)&server_addr, sizeof(server_addr));


	if (listen(tcp_fd, 3) < 0){
		perror("Listen failed");
		close(tcp_fd);
		close(recv_fd);
		exit(EXIT_FAILURE);
	}


	FD_ZERO(&read_fds);
	FD_SET(tcp_fd, &read_fds);
	FD_SET(recv_fd, &read_fds);

	//Select do wykrycia czy przychodzi polaczenie TCP(logowanie) czy UDP(czat)
	max_fd = (tcp_fd > recv_fd) ? tcp_fd : recv_fd;


	while (1){
		fd_set temp_fds = read_fds;
		int activity = select(max_fd + 1, &temp_fds, NULL, NULL, NULL);

		if(activity < 0){
			perror("select error");
			break;
		}

		if(FD_ISSET(tcp_fd, &temp_fds)){
			//polaczenie tcp
			if((client_fd = accept(tcp_fd, (struct sockaddr*)&tcp_addr, &addr_len)) < 0){
				perror("Accept Failed");
			}

			int x;
			x = recv(client_fd, buffer_tcp, BUFFER_SIZE, 0);
			if(x > 0){
				buffer_tcp[x-1] = '\0';
			}

			if(strcmp(buffer_tcp, PASSWORD) == 0){
				char bufor_tymczasowy[BUFFER_SIZE];
				//Bufor tymczasowy, zeby oddzielic dwa recv()
				sprintf(bufor_tymczasowy,"nic");
				if(send(client_fd, bufor_tymczasowy, BUFFER_SIZE, 0)<0){
					perror("blad");
          		          	exit(EXIT_FAILURE);
				}



				// otrzymywanie flagi
				ssize_t recv_test = recv(client_fd, buffer_tcp, BUFFER_SIZE, 0);
				if(recv_test > 0){
				buffer_tcp[recv_test] = '\0';
				}
				int num = atoi(buffer_tcp);



		                sprintf(wybor, "%d", num);
                
	
				load(buffer_tcp, sizeof(buffer_tcp), wybor);
				//sprintf(buffer_tcp,"maddd");
        		        if(send(client_fd, buffer_tcp, strlen(buffer_tcp), 0) < 0){
                			perror("blad wysylania");
                			exit(EXIT_FAILURE);
                		}


               			x = recv(client_fd, bufor_tymczasowy, BUFFER_SIZE, 0);
				if(x > 0){
					buffer_tcp[x-1] = '\0';
				}


				sprintf(buffer_tcp,"%ld", num+3758096384);
				if(send(client_fd, buffer_tcp, strlen(buffer_tcp), 0) < 0){
					perror("blad wysylania");
					exit(EXIT_FAILURE);
				}

			}
			close(client_fd);
		}


		//Polaczenie UDP
		if(FD_ISSET(recv_fd, &temp_fds)){
			n = recvfrom(recv_fd, buffer, BUFFER_SIZE, 0,  (struct sockaddr *)&client_addr, &addr_len);
			if(n < 0){
				perror("Blad odbierania");
				exit(EXIT_FAILURE);
			}
			buffer[n] = '\0';
			char buffer18[BUFFER_SIZE];
			sscanf(buffer,"%d;%s",&flaga,buffer2);
                        sprintf(wybor, "%d", flaga);

                        save(buffer2, wybor);
			group_addr.sin_addr.s_addr = htonl(3758096384+flaga);
			if(sendto(send_fd, buffer2, strlen(buffer2), 0, (const struct sockaddr *)&group_addr, addr_len) < 0){
				perror("Blad wysylania");
				exit(EXIT_FAILURE);
			}
		}
	}
	close(recv_fd);
	close(send_fd);
	close(tcp_fd);
	return 0;
}
