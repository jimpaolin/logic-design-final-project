// don't touch green, it will minus tour life, and add your time
// touch blue, it will stop the clock
//
//
// the "over table"
// 0~39 blue
// 40~59 green
// 60~99 red
module fallingObject(
   input CLK, clear, up, down, left, right,
	output reg A,B,C,D,E,F,G, 
	output reg COM3, COM4,
   output reg [7:0] position_R,position_G , position_B,
   output reg [2:0] S,
	output reg [3:0] LED,
	output reg EN,
	output reg beep
);
	//They are temp
	wire [7:0] p_green;
	wire [2:0] s_green;
	wire [7:0] p_blue;
	wire [2:0] s_blue;
	wire [7:0] p_red;
	wire [2:0] s_red;
	
	
	reg touch;//whether red touches blue or green
	reg add; //add time, bad
	reg minus;//minus time, good
   reg [1:0]count1;// for updating the 8*8 LED
	reg [2:0]count2; // for game over
	reg [3:0]life;// movingObject_red's life is 4 in the beginning.
	
	reg [3:0]time_score_COM3; // to store time in COMM3
	reg [3:0]time_score_COM4; // to store time in COMM4
	reg COM_choice; // choose COMM3 or COMM4
	
	reg finish;
	
	divfreq_time F1(CLK, CLK_time);
	divfreq F0(CLK, CLK_div);
	divfreq_updateClock F2(CLK, CLK_updateClock);

   initial begin
		position_R = 8'b01111111;
		position_G = 8'b11111111;
		position_B = 8'b11111111;
		EN = 1;
		count1 = 2'b0;
		count2 = 0;
		life = 4;
		touch = 0;
		add = 0;
		minus = 0;
		time_score_COM3 = 0;
		time_score_COM4 = 0;
		COM3 = 1;
		COM4 = 1;
		COM_choice = 1;
		finish = 0;
		beep = 0;
   end
	
	
	movingObject_red MRed(CLK, clear, up, down, left, right, p_red, s_red);
	fallingObject_blue FBlue(CLK, clear, p_blue, s_blue);// stop clock, good
	fallingObject_gree FGreen( CLK, clear, p_green, s_green); // add time, bad
	
	reg [3:0]a;
	// update clock
	always @(posedge CLK_updateClock)begin
		COM_choice = ~COM_choice;
		if(COM_choice) begin
			a = time_score_COM4;
			COM4 = 0;
			COM3 = 1;
		end
		else begin
			a = time_score_COM3;
			COM4 = 1;
			COM3 = 0;
		end
		case(a)
				0: {A,B,C,D,E,F,G}= 7'b0000001;
				1: {A,B,C,D,E,F,G}= 7'b1001111;
				2: {A,B,C,D,E,F,G}= 7'b0010010;
				3: {A,B,C,D,E,F,G}= 7'b0000110;
				4: {A,B,C,D,E,F,G}= 7'b1001100;
				5: {A,B,C,D,E,F,G}= 7'b0100100;
				6: {A,B,C,D,E,F,G}= 7'b0100000;
				7: {A,B,C,D,E,F,G}= 7'b0001111;
				8: {A,B,C,D,E,F,G}= 7'b0000000;
				9: {A,B,C,D,E,F,G}= 7'b0000100;
		endcase
	end
	
	// clock starts to run
	always @(posedge CLK_time, posedge clear) begin
		if(clear) begin
			time_score_COM3 = 0;
			time_score_COM4 = 0;
		end
		else begin
			if(finish)begin
				time_score_COM4 = time_score_COM4;
				time_score_COM3 = time_score_COM3;
			end
			else begin
				if(add)begin
					time_score_COM4 = time_score_COM4 + 3;
				end
				if(minus)begin
					if(time_score_COM4 < 10)begin
						time_score_COM4 = 0;
					end
					else begin
						time_score_COM4 = time_score_COM4 - 2;
					end
				end
				if(time_score_COM4 < 9) begin
					time_score_COM4 = time_score_COM4 + 1;
				end
				else if(time_score_COM4 != 9 && time_score_COM3 != 9)
				begin
					time_score_COM3 = time_score_COM3 + 1;
					time_score_COM4 = 0;
				end
				else begin
					time_score_COM4 = 9;
					time_score_COM3 = 9;
				end
			end
		end
	end
	
	//update all device
   always @(posedge CLK_div, posedge clear) begin
		if(clear) begin
			S = s_blue;
			position_B = ~p_blue;
			S = s_green;
			position_G = ~p_green;
			S = s_red;
			position_R= ~p_red;
			finish = 0;
			beep = 0;
		end
		else begin
			if(life == 0 || position_R == 8'b01111111)begin
				finish = 1;
			end
			if(finish)begin
				if(life == 0)begin
					LED = 4'b0000;
					position_G = 8'b11111111;
					position_B = 8'b11111111;
					if(count2 == 0)
						position_R = 8'b11101101;
					else if(count2 == 1)
						position_R = 8'b01010011;
					else if(count2 == 2)
						position_R = 8'b00010111;
					else if(count2 == 3)
						position_R = 8'b01001111;
					else if(count2 == 4)
						position_R = 8'b01111111;
					else if(count2 == 5)
						position_R = 8'b00000001;
					else if(count2 == 6)
						position_R = 8'b01101101;
					else if(count2 == 7)
						position_R = 8'b11101001;
				end
				else if(time_score_COM3 < 3)begin
					position_R = 8'b11111111;
					position_G = 8'b11111111;
					if(count2 == 0)
						position_B = 8'b11111111;
					else if(count2 == 1)
						position_B = 8'b11111110;
					else if(count2 == 2)
						position_B = 8'b10001101;
					else if(count2 == 3)
						position_B = 8'b11111011;
					else if(count2 == 4)
						position_B = 8'b11111011;
					else if(count2 == 5)
						position_B = 8'b10001101;
					else if(count2 == 6)
						position_B = 8'b11111110;
					else if(count2 == 7)
						position_B = 8'b11111111;
				end
				else if(time_score_COM3 < 5)begin
					position_R = 8'b11111111;
					position_B = 8'b11111111;
					if(count2 == 0)
						position_G = 8'b11111111;
					else if(count2 == 1)
						position_G = 8'b11111101;
					else if(count2 == 2)
						position_G = 8'b10001101;
					else if(count2 == 3)
						position_G = 8'b11111101;
					else if(count2 == 4)
						position_G = 8'b11111101;
					else if(count2 == 5)
						position_G = 8'b10001101;
					else if(count2 == 6)
						position_G = 8'b11111101;
					else if(count2 == 7)
						position_G = 8'b11111111;
				end
				else begin
					position_G = 8'b11111111;
					position_B = 8'b11111111;
					if(count2 == 0)
						position_R = 8'b11111111;
					else if(count2 == 1)
						position_R = 8'b11111011;
					else if(count2 == 2)
						position_R = 8'b10001101;
					else if(count2 == 3)
						position_R = 8'b11111110;
					else if(count2 == 4)
						position_R = 8'b11111110;
					else if(count2 == 5)
						position_R = 8'b10001101;
					else if(count2 == 6)
						position_R = 8'b11111011;
					else if(count2 == 7)
						position_R = 8'b11111111;
				end
				if(count2 < 8)begin
					S = count2;
					count2 = count2 + 1;
					beep = 1;	
				end
				
			end
			else begin
				// update 8*8LED
				count1 = count1 + 1;
				if(count1 > 2) begin
					count1 = 0;
				end
				if(count1 == 0) begin
					S = s_red;
					position_R = ~p_red;
					//temp_red = ~p_red;
					position_G = 8'b11111111;
					position_B = 8'b11111111;
				end
				else if(count1 == 1)begin
					S = s_blue;
					position_R = 8'b11111111;
					position_G = 8'b11111111;
					position_B = ~p_blue;
				end
				else if(count1 == 2) begin
					S = s_green;
					position_R = 8'b11111111;
					position_G = ~p_green;
					position_B = 8'b11111111;
				end
				
				// add time
				if((p_red==((~p_green)+2'b11))&&(s_red==s_green))begin
					add = 1;
					minus = 0;
					touch = 1;
				end
				
				//stop clock
				else if((p_red==((~p_blue)+2'b11))&&(s_red==s_blue))begin
					add = 0;
					minus = 1;
					touch = 1;
				end
				
				// update LED
				else begin
					touch = 0;
					if(life == 0)begin
						LED = 4'b0000;
					end
					else if(life == 1)begin
						LED = 4'b0001;
					end
					else if(life == 2)begin
						LED = 4'b0011;
					end
					else if(life == 3)begin
						LED = 4'b0111;
					end
					else if(life == 4)begin
						LED = 4'b1111;
					end
				end
			end
		end
	end
	always @(posedge touch, posedge clear) begin
		if(clear) begin
			life = 4;
		end
		else begin
			if(finish)begin
				life = life;
			end
			else begin
				if(add) begin
					if(life > 0)begin
						life = life - 1;
					end
				end
			end
		end
	end
endmodule

module movingObject_red(
	input CLK, clear, up, down, left, right, 
	output reg [7:0]pp, 
	output reg [2:0]ss
);
	reg [7:0]pp_temp;
	reg [2:0]ss_temp;
	
	initial begin
		pp = 8'b00000001;
		ss = 4;
		pp_temp = 8'b00000001;
		ss_temp = 4;
	end
	divfreq_red(CLK, CLK_div); 

	always @(posedge CLK_div) begin
		if(clear) begin
			pp = 8'b00000001;
			ss = 4;
			pp_temp = 8'b00000001;
			ss_temp = 4;
		end
		else begin
			if(left) begin
				if(ss != 0) begin
					ss<=ss-1;
					pp<=pp;
				end
				else begin
					ss<=0;
					pp<=pp;
				end
			end
		
			else if(right) begin
				if(ss != 7) begin
					ss<=ss+1;
					pp<=pp;
				end
				else begin
					ss<=7;
					pp<=pp;
				end
			end
	
			else if(up) begin
				if(pp != 8'b10000000) begin
					ss<=ss;
					pp<= pp << 1;
				end
				else begin
					ss<=ss;
					pp<=8'b10000000;
				end
			end
			
			else if(down) begin
				if(pp != 8'b00000001) begin
					ss<=ss;
					pp<= pp >> 1;
				end
				else begin
					ss<=ss;
					pp<=8'b00000001;
				end
			end
		end
	end
endmodule

module divfreq_red(input CLK, output reg CLK_div); 
	reg [24:0] Count; 
	always @(posedge CLK) 
		begin 
			if(Count > 2500000) 
				begin 
					Count <= 25'b0; CLK_div <= ~CLK_div; 
				end 
			else 
				Count <= Count + 1'b1; 	
		end 
endmodule 


module fallingObject_blue(
	 input CLK, clear,
	 output reg [7:0] pp, 
	 output reg [2:0] ss
);
	divfreq_UAD_blue F_UAD0(CLK, CLK_div_UAD);
   divfreq_LAR_blue F_LAR0(CLK, CLK_div_LAR);
	reg [24:0] cnt;
	initial begin
		cnt = 25'd0;
		pp = 8'b11000000;
		ss = 3'b000;
	end
	// left and right
   always@(posedge CLK_div_LAR) begin
		if(cnt > 250000)
			cnt = 25'd0;
      else
         cnt = cnt + 1;
   end
   // up and down
   always@(posedge CLK_div_UAD, posedge clear) begin
		if(clear) begin
			pp = 8'b11000000;
			ss = cnt % 8;
      end
      else begin
			pp = pp >> 1;
         if(pp == 8'b00000000) begin
				pp= 8'b11000000;
            ss = cnt %8;
         end
		end
	 end
endmodule

module divfreq_LAR_blue(input CLK, output reg CLK_div_LAR);
   reg [24:0]count;

   always @(posedge CLK) begin
       if(count > 123456) begin
           count = 0;
           CLK_div_LAR = ~CLK_div_LAR;
       end
       else 
           count = count + 1;
   end
endmodule


module divfreq_UAD_blue(input CLK, output reg CLK_div_UAD);
   reg [24:0] count;
   always @(posedge CLK) begin
       if(count > 2500000) begin
           count = 0;
           CLK_div_UAD = ~CLK_div_UAD;
       end
       else 
           count = count + 1;
   end
endmodule






module fallingObject_gree(
	 input CLK, clear,
	 output reg [7:0] pp, 
	 output reg [2:0] ss
);
	divfreq_UAD_gree F_UAD0(CLK, CLK_div_UAD);
   divfreq_LAR_gree F_LAR0(CLK, CLK_div_LAR);
	reg [24:0] cnt;
	initial begin
		cnt = 25'd0;
		pp = 8'b11000000;
		ss = 3'b000;
	end
	// left and right
   always@(posedge CLK_div_LAR) begin
		if(cnt > 250000)
			cnt = 25'd0;
      else
         cnt = cnt + 1;
   end
   // up and down
   always@(posedge CLK_div_UAD, posedge clear) begin
		if(clear) begin
			pp = 8'b11000000;
			ss = cnt % 8;
      end
      else begin
			pp = pp >> 1;
         if(pp == 8'b00000000) begin
				pp= 8'b11000000;
            ss = cnt %8;
         end
		end
	 end
endmodule




module divfreq_LAR_gree(input CLK, output reg CLK_div_LAR);
   reg [24:0]count;

   always @(posedge CLK) begin
       if(count > 8445171) begin
           count = 0;
           CLK_div_LAR = ~CLK_div_LAR;
       end
       else 
           count = count + 1;
   end
endmodule


module divfreq_UAD_gree(input CLK, output reg CLK_div_UAD);
   reg [24:0] count;
   always @(posedge CLK) begin
       if(count > 2000000) begin
           count = 0;
           CLK_div_UAD = ~CLK_div_UAD;
       end
       else 
           count = count + 1;
   end
endmodule


module divfreq(input CLK, output reg CLK_div_UAD);
   reg [24:0] count;
   always @(posedge CLK) begin
       if(count > 50000) begin
           count = 0;
           CLK_div_UAD = ~CLK_div_UAD;
       end
       else 
           count = count + 1;
   end
endmodule

module divfreq_time(input CLK, output reg CLK_time);
	reg [24:0] count;
   always @(posedge CLK) begin
       if(count > 25000000) begin
           count = 0;
           CLK_time = ~CLK_time;
       end
       else 
           count = count + 1;
   end

endmodule

module divfreq_updateClock(input CLK, output reg CLK_updateClock);
	reg [24:0] count;
   always @(posedge CLK) begin
       if(count > 50000) begin
           count = 0;
           CLK_updateClock = ~CLK_updateClock;
       end
       else 
           count = count + 1;
   end

endmodule
