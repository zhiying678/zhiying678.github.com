---
layout: post
title: About winpcap setnonblock function under win7
description: find some tissue using setnonblock() function under win7
category: blog
---

#setnonblock() in winpcap under win7
---

###My develop environment:
- 32bit win7 6.1.7601 Service Pack 1 Build 7601
- WinPcap version 4.1.3 (packet.dll version 4.1.0.2980), based on libpcap version
1.0 branch 1_0_rel0b (20091008)
- Microsoft Visual Studio 2010 版本 10.0.40219.1 SP1Rel

in the winpcap official development doc,it write
> ```c
> int pcap_setnonblock	(	pcap_t * 	p,
int 	nonblock,
char * 	errbuf	 
)
> ```			
Switch between blocking and nonblocking mode.
pcap_setnonblock() puts a capture descriptor, opened with pcap_open_live(), into "non-blocking" mode, or takes it out of "non-blocking" mode, depending on whether the nonblock argument is non-zero or zero. It has no effect on "savefiles". If there is an error, -1 is returned and errbuf is filled in with an appropriate error message; otherwise, 0 is returned. In "non-blocking" mode, an attempt to read from the capture descriptor with pcap_dispatch() will, if no packets are currently available to be read, return 0 immediately rather than blocking waiting for packets to arrive. pcap_loop() and pcap_next() will not work in "non-blocking" mode.


&ensp;&ensp;however,I find this function also has an effect on pcap_loop() and pcap_next_ex().
&emsp;when I open a device with paramater ***timeout*** set to 0 which means a following read routine will never return until the buffer is full.
&emsp;
the experiment result:

* pcap_setnonblock(fp,1,errbuf)  //read will return immeditealy when a packet arrive

* pcap_setnonblock(fp,0,errbuf))// read will not return until the buffer if full.

the **read** in the above two lines denote *pcap_loop()* 's *callback* function or *pcap_next_ex()* function,both OK.

my demo code below:

```c
#include<pcap.h>
#include <stdlib.h>
#include <stdio.h>
#define LINE_LEN 16

unsigned char sendbuf[]={
	//0xff,0xff,0xff,0xff,0xff,0xff,
	//0x00,0x23,0x89,0x32,0xf4,0x26,
	0xb8,0x88,0xe3,0xf6,0x57,0xf3,
	0xB8,0x88,0xE3,0xF6,0x57,0xF2,
	0x88,0x8e,0x01,0x00,
	0x00,0x37,0x02,0x1e,0x00,0x37,0x01,0x15,
	0x04,0xdb,0xe7,0x93,0x38,0x06,0x07,0x54,
	0x46,0x34,0x6b,0x59,0x41,0x51,0x71,0x49,
	0x6a,0x38,0x2b,0x48,0x45,0x4d,0x67,0x4f,
	0x6b,0x70,0x59,0x4e,0x6f,0x52,0x43,0x72,
	0x76,0x51,0x3d,0x20,0x20,0x32,0x30,0x30,
	0x37,0x39,0x39,0x30,0x31,0x35,0x32,0x31,
	0x30
};
unsigned char recvbuf[1024];

pcap_if_t *alldevs, *d;
pcap_t *fp;
void packet_handler(u_char *param, const struct pcap_pkthdr *header, const u_char *pkt_data)
{
	int static pktcnt=0;
	int pkt_size=header->len;
	int pkt_size2=header->caplen;

	memcpy(recvbuf,pkt_data,pkt_size2);
	//recvbuf[14]++;
	//if (pcap_sendpacket(fp, recvbuf, pkt_size2 /* size */) != 0)
	//{
	//	printf("\nError sending the packet: %s\n", pcap_geterr(fp));
	//	//return;
	//}

	/* print pkt timestamp and pkt len */
	//printf("%ld:%ld (%ld)\n", header->ts.tv_sec, header->ts.tv_usec, header->len);          
	printf("%ld:%ld (%ld)\tnum=%ld\n", header->ts.tv_sec, header->ts.tv_usec, header->len,++pktcnt);          

	/* Print the packet */
	//for (i=1; (i < header->caplen + 1 ) ; i++)
	//{
	//	printf("%.2x ", pkt_data[i-1]);
	//	if ( (i % LINE_LEN) == 0) printf("\n");
	//}
	printf("\n\n");  
}
int main(int argc,char ** argv)
{

	struct bpf_program localfilter;
	u_int inum, i=0,isndcnt=0;
	char errbuf[PCAP_ERRBUF_SIZE];
	int res=0;
	struct pcap_pkthdr *header;
	const u_char *pkt_data;
	printf("%s\n",pcap_lib_version());		
	printf("pktdump_ex: prints the packets of the network using WinPcap.\n");
	printf("   Usage: pktdump_ex [-s source]\n\n"
		"   Examples:\n"
		"      pktdump_ex -s file://c:/temp/file.acp\n"
		"      pktdump_ex -s rpcap://\\Device\\NPF_{C8736017-F3C3-4373-94AC-9A34B7DAD998}\n\n");

	if(argc < 3)
	{

		printf("\nNo adapter selected: printing the device list:\n");
		/* The user didn't provide a packet source: Retrieve the local device list */
		if (pcap_findalldevs_ex(PCAP_SRC_IF_STRING, NULL, &alldevs, errbuf) == -1)
		{
			fprintf(stderr,"Error in pcap_findalldevs_ex: %s\n", errbuf);
			return -1;
		}

		/* Print the list */
		for(d=alldevs; d; d=d->next)
		{
			printf("%d. %s\n    ", ++i, d->name);

			if (d->description)
				printf(" (%s)\n", d->description);
			else
				printf(" (No description available)\n");
		}

		if (i==0)
		{
			fprintf(stderr,"No interfaces found! Exiting.\n");
			return -1;
		}

		printf("Enter the interface number (1-%d):",i);
		scanf_s("%d", &inum);

		if (inum < 1 || inum > i)
		{
			printf("\nInterface number out of range.\n");

			/* Free the device list */
			pcap_freealldevs(alldevs);
			return -1;
		}

		/* Jump to the selected adapter */
		for (d=alldevs, i=0; i< inum-1 ;d=d->next, i++);

		/* Open the device */
		//
		//if ( (fp= pcap_open(d->name,
		//	65535 /*snaplen*/,
		//	PCAP_OPENFLAG_PROMISCUOUS /*flags*/,
		//	5000 /*read timeout*/,
		//	NULL /* remote authentication */,
		//	errbuf)
		//	) == NULL)
		/*obsolated usage*/
		if ( (fp= pcap_open_live(d->name,
			65535 /*snaplen*/,
			PCAP_OPENFLAG_PROMISCUOUS /*flags*/,
			0 /*read timeout*/,
			errbuf)
			) == NULL)
		{
			fprintf(stderr,"\nError opening adapter\n");
			return -1;
		}
		if(pcap_compile(fp,&localfilter,"ether proto 0x888e",1,0xffffff)==0)
		{
			printf("set filter OK!\n");
		}
		else
		{
			printf("set filter error!\n");
		}
		pcap_setfilter(fp,&localfilter);
		//pcap_compile(fp,&localfilter,"ether proto 0x8838",1,0xffffff);// test none block
		//pcap_compile(fp,&localfilter,"ether proto 0x0800",1,0xffffff);
		////pcap_compile(fp,&localfilter,"ether proto 0x0888e and ether host b8:88:e3:f6:57:f3",1,0xffffff);//ether dst?

		if(pcap_setbuff(fp,20*1024*1024)!=0)//20M
		{
			printf("set buffer error!\n");
		}
		else
		{
			printf("set buffer right!\n");
		}
		if(pcap_setnonblock(fp,1,errbuf))  //this will return immeditealy when a packet arrive
		//if(pcap_setnonblock(fp,0,errbuf))// this will not return until the buffer if full.
		{
			printf("set nonblock error!\n");
		}
		else
		{
			printf("set nonblock right!\n");
		}

		//if(pcap_setmintocopy(fp,100)!=0)
		//{
		//	printf("set buffer error!\n");
		//}
		//else
		//{
		//	printf("set buffer right!\n");
		//}

	}
	//else 
	//{
	//	// Do not check for the switch type ('-s')
	//	if ( (fp= pcap_open(argv[2],
	//		65535 /*snaplen*/,
	//		PCAP_OPENFLAG_PROMISCUOUS /*flags*/,
	//		  /*read timeout*/,
	//		NULL /* remote authentication */,
	//		errbuf)
	//		) == NULL)
	//	{
	//		fprintf(stderr,"\nError opening source: %s\n", errbuf);
	//		return -1;
	//	}
	//}


	scanf("%d",&inum);

	//test blob
	//scanf("%d",&inum);
	//isndcnt=0;
	//while(inum!=5)
	//{

	//	if (pcap_sendpacket(fp, sendbuf, sizeof(sendbuf) /* size */) != 0)
	//	{
	//		printf("\nError sending the packet: %s\n", pcap_geterr(fp));
	//		//return;
	//	}
	//	printf("send%d\n",isndcnt++);
	//	sendbuf[15]=isndcnt;
	//	scanf("%d",&inum);
	//}


	/* Read the packets */
	pcap_freealldevs(alldevs);

	/* start the capture */
	//pcap_loop(fp, 0, packet_handler, NULL);
	//pcap_dispatch(fp,-1,packet_handler,NULL);


	while((res = pcap_next_ex( fp, &header, &pkt_data)) >= 0)
	{
		int pkt_size=header->len;
		int pkt_size2=header->caplen;
		if(res == 0)
			/* Timeout elapsed */
			continue;

		//memcpy(recvbuf,pkt_data,pkt_size2);
		//recvbuf[14]++;
		//if (pcap_sendpacket(fp, recvbuf, pkt_size2 /* size */) != 0)
		//{
		//	printf("\nError sending the packet: %s\n", pcap_geterr(fp));
		//	//return;
		//}

		/* print pkt timestamp and pkt len */
		printf("%ld:%ld (%ld)\tnum=%ld\n", header->ts.tv_sec, header->ts.tv_usec, header->len,recvbuf[15]);          

		/* Print the packet */
		//for (i=1; (i < header->caplen + 1 ) ; i++)
		//{
		//	printf("%.2x ", pkt_data[i-1]);
		//	if ( (i % LINE_LEN) == 0) printf("\n");
		//}

		printf("\n\n");     
	}




	//if(res == -1)
	//{
	//	fprintf(stderr, "Error reading the packets: %s\n", pcap_geterr(fp));
	//	return -1;
	//}

	printf("dispatch over\n");
	scanf("%d",&inum);
	return 0;
}
```



以防自己以后看不懂，汉语解释一下：
1. pcap_setnonblock()函数，当第二个参数设为非0时表示非阻塞模式，来数据包立即就能读到。
2. 若设为0表示阻塞模式，此时
- 若pcap_open()设置timeout为0，只有当缓冲区满时才会返回，一次处理多个数据包。
- 若pcap_open()设置timeout非0，只有缓冲区满或超时时间到达时，才会返回。


[zhiying678]:    http://zhiying678.github.io  "zhiying678"