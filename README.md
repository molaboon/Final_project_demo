# Final_project_demo
組員 108321010 陳宇鴻 108321008蔡宜哲

Demo video:
https://www.youtube.com/watch?v=vMCLttaGje8&ab_channel=%E9%99%B3%E5%AE%87%E6%96%87

Code:

	module finalpro(
	input CLK, reset, start,restart,level1,level2,
	output reg [0:27] led,
	//output reg [2:0] life,
	input left, right,
	input throw,
	//output testLED,
	output reg a,b,c,d,e,f,g
	);

	reg [7:0]blockFirst =  8'b11111111;
	reg [7:0]blockSecond = 8'b11111111;
	reg [7:0]block_stone = 8'b00000000;
	reg [2:0]ball_position;	// 球  位置
	reg [2:0]ball_y_position; // 球 y 座標
	reg [2:0]stone_pos;
	reg [3:0]ball_count;
	
	integer horizonPosition;

	reg handsOn; 				// bool，紀錄球現在丟出去了沒
	reg gameoverflag;
	reg r;
	reg r7;
	
	initial
	begin
		// 歸零，重製 8x8 LED
		led[0:23] = 24'b11111111111111111111;
		led[27] = 1;
		led[24:26] = 3'b000;
		ball_position = 3'b011;		// 預設在 x=3 的位置
		ball_y_position = 3'b010;	// 預設在 y=1 的位置
		stone_pos =3'b100;
		handsOn = 1;					// 預設為 為丟出狀態
		gameoverflag=0;
		ball_count=4'b0000;
		horizonPosition = 0;			// 預設為 正中間方向
		r=0;
		r7=0;
		//gameoverflag = 0;

	end
	// 開始所有除頻器
	divfreq F(CLK, divclk);
	buttondivfreq BT(CLK, buttonclk);
	
	integer ballTime;
	// 判斷 所有操作
	always @(posedge buttonclk )
	begin
		if(restart)
		begin
			gameoverflag=0;
			blockFirst =  8'b11111111;
			blockSecond = 8'b11111111;
			//led[16:23] = {~blockFirst[row], ~blockSecond[row], 6'b111111};
			ball_position <= 3'b010;		// 預設在 x=3 的位置
			ball_y_position <= 3'b010;	// 預設在 y=1 的位置
			handsOn = 1;		
			ball_count<=0;		// 預設為 為丟出狀態
		end//end restart
		else
		begin 
		if(start)
		begin
			if(reset||r||r7)
			begin
				ball_position <= 3'b010;		// 預設在 x=3 的位置
				ball_y_position <= 3'b010;	// 預設在 y=1 的位置
				handsOn = 1;				// 預設為 為丟出狀態
				r=0;
				r7=0;
					// 預設為 向上
				horizonPosition = 0;		// 預設為 正中間方向
			end
			// 判斷 向左
			if(left)
				if(ball_position>0)
				begin
					if(handsOn==1)ball_position <= ball_position-1;
				end
			
			// 判斷 向右
			if(right)
				if(ball_position<7)
				begin
					if(handsOn==1)ball_position <= ball_position+1;
				end
					
			// 判斷 丟出球
			if(throw)
				if(handsOn)
				begin
					r=0;
					handsOn = 0;
					if(ball_count<9)
						begin
							ball_count<=ball_count+1'b1;
						end
					else
						begin
							ball_count<=0;
						end
				end
			// 下方操作球的運行
			// 除頻用
			if(ballTime<2)
				ballTime <= ballTime+1;
			else
			//開始判斷球的行進
			begin
				ballTime <= 0;
				if(handsOn==0)	// 如果是丟出去的狀態才移動
					begin
						if(ball_y_position<=7)
							begin
								ball_y_position<=ball_y_position+1;
								//check stone
								if(ball_y_position==3&&(stone_pos==ball_position||stone_pos==ball_position-1))
									begin
										r=1;
									end
								//check block
								if(ball_y_position==6&&blockSecond[ball_position]==1)
									begin
									ball_y_position<=ball_y_position;
									blockSecond[ball_position]=0;
									blockSecond[ball_position-1]=0;
									blockSecond[ball_position+1]=0;
									blockFirst[ball_position]=0;
									r=1;
									end
								if(ball_y_position==7&&blockFirst[ball_position]==1)
									begin
									ball_y_position<=ball_y_position;
									blockFirst[ball_position]=0;
									blockFirst[ball_position+1]=0;
									blockFirst[ball_position-1]=0;
									r7=1;
									end
								//gameoverflag
								if(blockFirst[0]==0&&blockFirst[1]==0&&blockFirst[2]==0&&blockFirst[3]==0&&blockFirst[4]==0&&blockFirst[5]==0&&blockFirst[6]==0&&blockFirst[7]==0&&blockSecond[0]==0&&blockSecond[1]==0&&blockSecond[2]==0&&blockSecond[3]==0&&blockSecond[4]==0&&blockSecond[5]==0&&blockSecond[6]==0&&blockSecond[7]==0)
									gameoverflag=1;
							end	
							
					end
			end
		end
	end//restart
	end//end of always buttonclk
	
	// 顯示用
	always @(posedge divclk)
	begin
		reg [0:2]row;
		reg count_digit_enable;
		reg [11:0] tmpcount;
		reg [10:0] tmptmp_count;
		reg [7:0] mode;
		reg [7:0] level_mid;
		// 跑 0~7 行
		if(row>=7)
			row <= 3'b000;
		else
			row <= row + 1'b1;
		
		// 設定這次要畫第 n 行
		led[24:26] = row;
		
		
		// 如果 gameoverflag ==1就畫圖
		if(gameoverflag)
		begin
			//led[0:23] = 24'b111111111111111111111111;
			if(row==0 || row==7) led[8:24] 		= 8'b11000011;
			else if(row==1 || row==6) led[8:24]	= 8'b10111101;
			else if(row==2 || row==5) led[8:24]	= 8'b01111110;
			else if(row==3 || row==4) led[8:24]	= 8'b01111110;
			else led[8:24]	= 8'b11111111;
		end
		else
		begin
			// 開始畫球 ( G )
			led[0:23]=24'b111111111111111111111111;
			if(handsOn)
				if(row==ball_position)		// 放在正中間
					led[8:15] = 8'b11111110;
				else
					led[8:15] = 8'b11111111;
			else
			begin
				if(row==ball_position)
				begin
					reg [7:0] map;
					case(ball_y_position)
						3'b000: map = 8'b11111110 ;
						3'b001: map = 8'b11111101 ;
						3'b010: map = 8'b11111011 ;
						3'b011: map = 8'b11110111 ;
						3'b100: map = 8'b11101111 ;
						3'b101: map = 8'b11011111 ;
						3'b110: map = 8'b10111111 ;
						3'b111: map = 8'b01111111 ; 
					endcase
					led[8:15] = map;
				end
				else
					led[8:15] = 8'b11111111;
			end//end of else

			//開始畫磚塊
			if(restart)
			begin
				led[16:23] = {~blockFirst[row], ~blockSecond[row], 6'b111111};
			end//end of restart
				led[16:23] = {~blockFirst[row], ~blockSecond[row], 6'b111111};
			
			if(start)
			begin
				if(tmpcount>500)
					begin
					tmpcount<=0;
					stone_pos<=stone_pos+1'b1;
					end	
				else
					begin
						tmpcount<=tmpcount+1'b1;
						case(stone_pos)
							3'b000: mode = 8'b11111100 ;
							3'b001: mode = 8'b11111001 ;
							3'b010: mode = 8'b11110011 ;
							3'b011: mode = 8'b11100111 ;
							3'b100: mode = 8'b11001111 ;
							3'b101: mode = 8'b10011111 ;
							3'b110: mode = 8'b00111111 ;
							3'b111: mode = 8'b01111110 ; 
						endcase
						led[4]=mode[row];
						
					end
			end
      //計算球數
			if(start)
			begin
			case(ball_count)
				4'b0000: {a,b,c,d,e,f,g}= 7'b0000001;
				4'b0001: {a,b,c,d,e,f,g}= 7'b1001111;
				4'b0010: {a,b,c,d,e,f,g}= 7'b0010010;
				4'b0011: {a,b,c,d,e,f,g}= 7'b0000110;
				4'b0100: {a,b,c,d,e,f,g}= 7'b1001100;
				4'b0101: {a,b,c,d,e,f,g}= 7'b0100100;
				4'b0110: {a,b,c,d,e,f,g}= 7'b0100000;
				4'b0111: {a,b,c,d,e,f,g}= 7'b0001111;
				4'b1000: {a,b,c,d,e,f,g}= 7'b0000000;
				4'b1001: {a,b,c,d,e,f,g}= 7'b0000100;
				default: {a,b,c,d,e,f,g}= 7'b1111111;
			endcase
			end
		end
	end //end of always	
	endmodule


	// 顯示用的除頻器
	module divfreq(input CLK, output reg CLK_div);
		reg[24:0] Count;
		always @(posedge CLK)
		begin
			if(Count>25000)
				begin
					Count <= 25'b0;
					CLK_div <= ~CLK_div;
				end
			else
				Count <= Count + 1'b1;
		end
	endmodule


	// 按鈕用的除頻器
	module buttondivfreq(input CLK, output reg CLK_div);
		reg[24:0] Count;
		always @(posedge CLK)
		begin
			if(Count>2500000)			// 20 Hz
				begin
					Count <= 25'b0;
					CLK_div <= ~CLK_div;
				end
			else
				Count <= Count + 1'b1;
		end
	endmodule
