#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <arpa/inet.h>
#include <unistd.h>
#include <sys/select.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netdb.h>

#define PORT_TCP 8000
#define PORT2 14
#define PORT 12345
#define BUFFER_SIZE 1024

int main(int argc, char *argv[]){
	int recv_fd, send_fd, tcp_fd;
	char buffer[BUFFER_SIZE], buffer2[BUFFER_SIZE], buffer3[BUFFER_SIZE], buffer_login[BUFFER_SIZE], buffer_new[BUFFER_SIZE];
	struct sockaddr_in server_addr, serverudp_addr, group_addr;
	struct ip_mreq mreq;
	struct addrinfo hints, *res;
	fd_set read_fds;
	int max_fd;
	int test1;


	if (argc != 2){
		printf("Uzycie: %s [adres serwera]\n", argv[0]);
		return 1;
	}
	char *adres_serwera = argv[1];




	//Wysylanie UDP
	if((recv_fd = socket(AF_INET, SOCK_DGRAM, 0)) < 0){
		perror("Blad tworzenia gniazda recv");
		exit(EXIT_FAILURE);
	}

	//Odbieranie UDP
        if((send_fd = socket(AF_INET, SOCK_DGRAM, 0)) < 0){
                perror("Blad tworzenia gniazda send");
                exit(EXIT_FAILURE);
        }

	//Komunikacja TCP
	if((tcp_fd = socket(AF_INET, SOCK_STREAM, 0)) < 0){
		perror("Blad tworzenia gniazda tcp");
		exit(EXIT_FAILURE);
	}


	//Konfiguracja getaddrinfo
	memset(&hints, 0, sizeof(hints));
	if(getaddrinfo(adres_serwera, NULL, NULL, &res) !=0){
		perror("getaddrinfo error");
		exit(EXIT_FAILURE);
	}



	//Adres serwera dla TCP
	memset(&server_addr, 0, sizeof(server_addr));
	server_addr.sin_family = AF_INET;
	server_addr.sin_port = htons(PORT_TCP);
	//server_addr.sin_addr.s_addr = inet_addr("10.0.2.5");  //Tu adres serwera jesli nie ma pliku DNS
	server_addr.sin_addr.s_addr = ((struct sockaddr_in *)res->ai_addr)->sin_addr.s_addr; //Zadziala jesli skonfigurowany /etc/hosts !

	//Adres serwera dla UDP
	memset(&serverudp_addr, 0, sizeof(serverudp_addr));
        serverudp_addr.sin_family = AF_INET;
        serverudp_addr.sin_port = htons(PORT2);
    	//serverudp_addr.sin_addr.s_addr = inet_addr("10.0.2.5");	//Tu adres serwera jesli nie ma pliku DNS
	serverudp_addr.sin_addr.s_addr = ((struct sockaddr_in *)res->ai_addr)->sin_addr.s_addr;

	freeaddrinfo(res);

	//Obsluga logowania
	printf("Wpisz login uzytkownika: ");
	fgets(buffer_login, BUFFER_SIZE, stdin);
        buffer_login[strcspn(buffer_login, "\n")] = '\0';


	printf("Wpisz haslo aby dolaczyc do serwera: ");
	fgets(buffer, BUFFER_SIZE, stdin);

	printf("Wpisz numer kanalu: ");
	fgets(buffer3, BUFFER_SIZE, stdin);
	int flaga = atoi(buffer3);


	

	if(connect(tcp_fd, (const struct sockaddr *)&server_addr, sizeof(server_addr)) < 0){
		perror("Blad connect");
		exit(EXIT_FAILURE);
	}

	//Wyslanie hasla
	if(send(tcp_fd, buffer, strlen(buffer), 0) < 0){
                perror("Blad wysylania");
                exit(EXIT_FAILURE);
	}

	
	//Bufer, zeby nie powtarzac sendow
	char bufer_tymczasowy[BUFFER_SIZE];
	sprintf(bufer_tymczasowy,"nic");
	ssize_t recv_len = recv(tcp_fd, bufer_tymczasowy, BUFFER_SIZE, 0);
	if (recv_len > 0){
        buffer[recv_len] = '\0';
        }



	//Wysylanie flagi, ktora posluzy serwerowi do odeslania poprawnego adresu pokoju
	if(send(tcp_fd, buffer3, BUFFER_SIZE, 0) < 0){
		perror("Blad wysylania");
		exit(EXIT_FAILURE);
	}



	recv_len = recv(tcp_fd, bufer_tymczasowy, BUFFER_SIZE, 0);
	if(recv_len > 0){
        buffer[recv_len] = '\0';
        }



    	//Odebranie historii
	recv_len = recv(tcp_fd, buffer, BUFFER_SIZE, 0);
	if(recv_len > 0){
        	buffer[recv_len] = '\0';
			printf("Historia czatu:\n%s\n", buffer);
        }






	if(send(tcp_fd, bufer_tymczasowy, BUFFER_SIZE, 0) < 0){
		perror("Blad wysylania");
		exit(EXIT_FAILURE);
	}


	//Odebranie adresu multicast od serwera
	recv_len = recv(tcp_fd, buffer2, BUFFER_SIZE, 0);
        if (recv_len > 0){
		buffer2[recv_len] = '\0';
        }

	//num bedzie uzyty w mreq do poldaczenia sie do adresu multicast
	long int num = atoll(buffer2);




	//Ustawienie grupy multicast
	memset(&group_addr, 0, sizeof(group_addr));
	group_addr.sin_family = AF_INET;
	group_addr.sin_port = htons(PORT);
	group_addr.sin_addr.s_addr = htonl(INADDR_ANY);

	mreq.imr_multiaddr.s_addr = htonl(num);
	mreq.imr_interface.s_addr = htonl(INADDR_ANY);


	if( bind(recv_fd, (struct sockaddr*)&group_addr, sizeof(group_addr)) < 0){
                perror("Bind error");
                close(recv_fd);
                exit(EXIT_FAILURE);
        }

	if( setsockopt(recv_fd, IPPROTO_IP, IP_ADD_MEMBERSHIP, &mreq, sizeof(mreq)) < 0){
		perror("Blad dolaczania do grupy multicastowej - Podales zle haslo");
		close(recv_fd);
		exit(EXIT_FAILURE);
	}


	//Uzycie select, aby polaczenie nie blokowalo sie czekajac na dane z klawiatury
	max_fd = recv_fd > fileno(stdin) ? recv_fd : fileno(stdin);

	while (1) {
		FD_ZERO(&read_fds);
		FD_SET(recv_fd, &read_fds);
		FD_SET(fileno(stdin), &read_fds);

		if (select(max_fd + 1, &read_fds, NULL, NULL, NULL) < 0){
			perror("Blad select");
			break;
		}
		//Odbior danych w recv
		if (FD_ISSET(recv_fd, &read_fds)){
			memset(buffer, 0, BUFFER_SIZE);
			ssize_t recv_len = recvfrom(recv_fd, buffer, BUFFER_SIZE, 0, NULL, NULL);
			if (recv_len > 0){
				buffer[recv_len] = '\0';
				printf("%s\n", buffer);
			}
		}


		//Obsluga klawiatury i wyslanie
		if (FD_ISSET(fileno(stdin), &read_fds)) {
			memset(buffer, 0, BUFFER_SIZE);
			if (fgets(buffer, BUFFER_SIZE, stdin) !=NULL) {
				buffer[strcspn(buffer, "\n")] = '\0';
				if(strcmp(buffer, "exit") == 0){
					printf("Zamykanie");
					break;
				}
				sprintf(buffer3, "%d;%s:%s", flaga,buffer_login, buffer);
				if(sendto(send_fd, buffer3, strlen(buffer3), 0, (const struct sockaddr *)&serverudp_addr, sizeof(serverudp_addr)) < 0){
					perror("Blad wysylania");
					exit(EXIT_FAILURE);
				}
			}
		}
	}
	close(send_fd);
	close(recv_fd);
	return 0;
}
