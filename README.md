# FTP-using-UDP-socket-server-client-C-programmingSection Operation :

	A: 	Modifications to ensure unicast addresses are bound: 
		We made sure that only unicast addresses are bound and did not use the broadcast address field from the get_ifi_info_plus 			function.
	
	B: 	Array of structures for sockets:
		The interfaces which we get from the get_ifi_info_plus function along with the socket are stored in an array of structures as 			follows:

			struct record 
			{
				int 			sockfd;
				char 			ipaddr[20],nwaddr[20],sbaddr[20];
			}re[10];

Section Adding Reliability to UDP: 

1: 	Modifications of code in section 22.5
		We included the declarations and definitions from the unprtt.h header file directly into our server code as follows. 

		RTT Values:
		
			RTT_RXTMIN		1000 			/* 1 second (in msec) */
			RTT_RXTMAX		3000			/* 3 seconds (in msec) */
			RTT_MAXNREXMT		12 			/* maximum amount of times to retransmit */

			int	 		rtt;			/* most recent rtt in msec */
			int 			srtt;			/* smoothed rtt in msec inflated by 8X */
			int 			rttvar; 		/* variance inflated by 4X */
			int 			rto;			/* current calculation to use in msec */
			int 			windowPing;		/* how long until sending the next window ping in msec */

		If the packet is retransmitted then we are not calculating its rtt value for sending ack. However, if the number 
		of acks received for a particular packet is less than four than we are using rtt calculations for sending the 
		corresponding ack.
	 	
	
2:Window and TCP mechanism:

	A:  	The ARQ sliding window technique is implemented using linked list as follows:
			struct packet
			{
				struct packet 			*next;			//pointer to next packet
				struct header 			pheader;		//header of the packet
				char 				body[512];		//body of the packet
				int 				p_ack;			//no. of acks recieved for a packet
				int 				pcount;			//no. of times a packet sent
				unsigned long long int 		timestamp; 		
			};
		This helps us keep a track of the current window capacity and maintaining the packets containing the header and body
		in which are to communicated.

	B:	The header structure included in the packet structure is defined as:
			struct header
			{
				int	 			seq_num;		//packet number
				int	 			ack;			//acknowledgment flag
				int	 			fin;			//fin flag
				int	 			receive_window;		//size of current window
				int				body_size;		//size of body
			};
		The structure header is copied into the sending buffer or from recieving buffer using memcpy function.(Generally 
		we are using HEADER size of 20 bytes and body size of 512 bytes).
	  
	C:  	Resending:
		We set the timer of itimer function by using the ITIMER flag to the value obtained from the RTT calculation.
		If the timer timeouts, then siglongjmp is used which helps in avoiding the race conditions and resends the signal.
	  
	 
	D:	Handling ACKs:
		Once we recieve an acknowledgement from the client about a particular packet, then packets are removed from the
		window by using the removep function till the sequence number we get from the header of recieved packet and then 
		send the packet of the sequence number. Adding on, when we recieve an ACk for a particular packet, we maintain and 
		update a counter flag. When this flag sets to a value equal to 4 then the server executes a fast retransmit of that 
		packet. Furthermore, there is also a flag which maintains the number of times a particular packet is being sent. If 
		this flag exceeds the maximum value of 12 then the server stops sending packets. 

3:	Last Packet:
		When the server sends the last packet then it sets the last flag and sends a fin signal by setting the fin variable 
		in the header structure. this is backed up by a timing out mechanism incase the packet is lost or the server does 
		not recieves an acknowledgement.Furthermore, the client sends an acknowledgement and a fin signal in response to the 
		fin signal obtained from the server and enters a wait state. If the client does not recieves any signal then it comes out of it 		wait state and exits.On the other hand, the server closes its connection as soon as it recieves an acknowledgement for all the 			packets.
 
  
 

