#include <sys/types.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

#define ERR_EXIT(msg) do{perror(msg);exit(-1);}while(0)

// 将数据转换成大写，toupper()是C标准库提供的
void upper(char* buf) {
    char *p = buf;
    while (*p) {
        *p = toupper(*p);
        ++p;
    }
}

int main() {
    int ret, clientfd, sockfd, n;
    struct sockaddr_in servaddr; // 套接字地址
    struct sockaddr_in cliaddr;
    struct sockaddr_in newaddr;

    socklen_t cliaddrlen, newlen;
    char buf[128];
    
    // 1.creat sockaddr
    puts("1.creat sockaddr");
    servaddr.sin_family = AF_INET;
    servaddr.sin_port = htons(8080); // 端口号也要转为网络字节序
    if (inet_aton("192.168.178.134", &servaddr.sin_addr) == 0) {
        fprintf(stderr, "Invalid address\n");
        exit(-1);
    }
    printf("selfsockaddr: %s:%d\n", inet_ntoa(servaddr.sin_addr), ntohs(servaddr.sin_port));
    
    // 2.creat socket
    puts("2.creat socket");
    sockfd = socket(AF_INET, SOCK_STREAM, 0);
    if (sockfd < 0) perror("socket");
    
    // 3.bind socket
    puts("3.bind socket");
    ret = bind(sockfd, (struct sockaddr*)&servaddr, sizeof(servaddr)); // 转为通用套接字
    if (ret < 0) perror("bind");

    // 4.listen
    puts("4.listen");
    ret = listen(sockfd, 5);
    if (ret < 0) ERR_EXIT("listen");
    
    // 5.accept connect
    puts("5.accept connect");
    cliaddrlen = sizeof(cliaddr);
    clientfd = accept(sockfd, (struct sockaddr*)&cliaddr, &cliaddrlen);
    if (clientfd < 0) ERR_EXIT("accept");
    printf("client fd: %d\n", clientfd);
    printf("clisockaddr: %s:%d\n", inet_ntoa(cliaddr.sin_addr), ntohs(cliaddr.sin_port));
    
    // 6.getsockname 打印新的 socket 绑定的套接字地址
    puts("6.getsockname");
    newlen = sizeof(newaddr);
    ret = getsockname(clientfd, (struct sockaddr*)&newaddr, &newlen);
    if (ret < 0) ERR_EXIT("getsockaddr");
    printf("newsockaddr: %s:%d\n", inet_ntoa(newaddr.sin_addr), ntohs(newaddr.sin_port));

    // 7.send data
    while (1) {
        // 从客户端读取数据，如果返回0表示对端关闭
        n = read(clientfd, buf, 63);
        if (n == 0) {
            puts("peer closed");
            break;
        }
        buf[n] = 0;
        // 将缓冲区中的数据转换为大写
        upper(buf);
        // 发送回客户端
        write(clientfd, buf, n);
    }
    close(clientfd);
    close(sockfd);

    return 0;
}
