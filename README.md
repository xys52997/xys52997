//#include "system.h"
#include "rc522.h"
#include "SysTick.h"
#include "usart.h"
#include "string.h"
#include "stm32f10x_spi.h"
#include "main.h"
//
// M1¿¨·ÖÎª16¸öÉÈÇø£¬Ã¿¸öÉÈÇøÓÉËÄ¸ö¿é£¨¿é0¡¢¿é1¡¢¿é2¡¢¿é3£©×é³É
// ½«16¸öÉÈÇøµÄ64¸ö¿é°´¾ø¶ÔµØÖ·±àºÅÎª£º0~63
// µÚ0¸öÉÈÇøµÄ¿é0£¨¼´¾ø¶ÔµØÖ·0¿é£©£¬ÓÃÓÚ´æ·Å³§ÉÌ´úÂë£¬ÒÑ¾­¹Ì»¯²»¿É¸ü¸Ä
// Ã¿¸öÉÈÇøµÄ¿é0¡¢¿é1¡¢¿é2ÎªÊý¾Ý¿é£¬¿ÉÓÃÓÚ´æ·ÅÊý¾Ý
// Ã¿¸öÉÈÇøµÄ¿é3Îª¿ØÖÆ¿é£¨¾ø¶ÔµØÖ·Îª:¿é3¡¢¿é7¡¢¿é11.....£©°üÀ¨ÃÜÂëA£¬´æÈ¡¿ØÖÆ¡¢ÃÜÂëBµÈ

/*******************************
*Á¬ÏßËµÃ÷£º
*1--SDA  <----->PA4
*2--SCK  <----->PA5
*3--MOSI <----->PA7
*4--MISO <----->PA6
*5--Ðü¿Õ
*6--GND <----->GND
*7--RST <----->PB0
*8--VCC <----->VCC
************************************/

/*È«¾Ö±äÁ¿*/
unsigned char CT[2];//¿¨ÀàÐÍ
unsigned char SN[4]; //¿¨ºÅ
u8 DATA[16];			//´æ·ÅÊý¾Ý
unsigned char RFID[16];			//´æ·ÅRFID
unsigned char card0_bit=0;
unsigned char card1_bit=0;
unsigned char card2_bit=0;
unsigned char card3_bit=0;
unsigned char card4_bit=0;
unsigned char total=0;
// ÕâUID¶¨ÒåÔÚÕâ²»ÖªµÀ¸ÉÉ¶ÓÃµÄ¡£¡£¡£ Ìæ»»³É×Ô¼º¿¨µÄUID
unsigned char card_0[4]= {165,50,05,00};
unsigned char card_1[4]= {105,102,100,152};
unsigned char card_2[4]= {208,121,31,57};
unsigned char card_3[4]= {176,177,143,165};
unsigned char card_4[4]= {5,158,10,136};
u8 KEY_A[6]= {0xff,0xff,0xff,0xff,0xff,0xff};
u8 KEY_B[6]= {0xff,0xff,0xff,0xff,0xff,0xff};
u8 AUDIO_OPEN[6] = {0xAA, 0x07, 0x02, 0x00, 0x09, 0xBC};
// ²âÊÔÓÃ 3Çø¿éÊý¾Ý
unsigned char RFID1[16]= {0x10,0x20,0x30,0x40,0x50,0x60,0xff,0x07,0x80,0x29,0x01,0x02,0x03,0x04,0x05,0x06};
unsigned char RFID2[16]= {0xff,0xff,0xff,0xff,0xff,0xff,0xff,0x07,0x80,0x29,0xff,0xff,0xff,0xff,0xff,0xff};
// ²âÊÔÓÃ 3Çø¿éÃÜÔ¿
u8 KEY_A1[6]= {0x10,0x20,0x30,0x40,0x50,0x60};
u8 KEY_A2[6]= {0x00,0x00,0x00,0x00,0x00,0x00};
u8 KEY_B1[6]= {0x01,0x02,0x03,0x04,0x05,0x06};
u8 KEY_B2[6]= {0x10,0x20,0x30,0x00,0x00,0x00};
u8 KEY_B3[6]= {0x01,0x02,0x03,0x00,0x00,0x00};
// ÖÃÁãÓÃ
unsigned char DATA0[16]= {0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00};
unsigned char DATA1[16]= {0x12,0x34,0x56,0x78,0x9A,0x00,0xff,0x07,0x80,0x29,0xff,0xff,0xff,0xff,0xff,0xff};
unsigned char status;
// 0x08 ¾ÍÊÇ2ÉÈÇø0Çø¿é£¨¼´µÚ9¿é£©
unsigned char addr=0x08;
// unsigned char addr=0x08;

#define   RC522_DELAY()  delay_us( 20 )

// ²âÊÔ³ÌÐò0£¬Íê³Éaddr¶ÁÐ´¶Á
void RC522_Handle(void)
{
    u8 i = 0;
    status = PcdRequest(PICC_REQALL,CT);//Ñ°¿¨

    // printf("\r\nstatus>>>>>>%d\r\n", status);

    if(status==MI_OK)// Ñ°¿¨³É¹¦
    {
        status=MI_ERR;
        status = PcdAnticoll(SN);// ·À³å×² »ñµÃUID ´æÈëSN
			
    }

    if (status==MI_OK)// ·À³å×²³É¹¦
    {
			
        status = MI_ERR;
        ShowID(SN); // ´®¿Ú´òÓ¡¿¨µÄIDºÅ UID
			
        
        // ÄÑµÀ¾ÍÊÇÎªÁË×ö¸öÅÐ¶ÏÂð¡£¡£¡£
        if((SN[0]==card_0[0])&&(SN[1]==card_0[1])&&(SN[2]==card_0[2])&&(SN[3]==card_0[3]))
        {
            card0_bit=1;
            printf("\r\nThe User is:card_0\r\n");
        }
        if((SN[0]==card_1[0])&&(SN[1]==card_1[1])&&(SN[2]==card_1[2])&&(SN[3]==card_1[3]))
        {
            card1_bit=1;
            printf("\r\nThe User is:card_1\r\n");
        }
        if((SN[0]==card_2[0])&&(SN[1]==card_2[1])&&(SN[2]==card_2[2])&&(SN[3]==card_2[3]))
        {
            card2_bit=1;
            printf("\r\nThe User is:card_2\r\n");
        }

        if((SN[0]==card_3[0])&&(SN[1]==card_3[1])&&(SN[2]==card_3[2])&&(SN[3]==card_3[3]))
        {
            card3_bit=1;
            printf("\r\nThe User is:card_3\r\n");
        }
        if((SN[0]==card_4[0])&&(SN[1]==card_4[1])&&(SN[2]==card_4[2])&&(SN[3]==card_4[3]))
        {
            card4_bit=1;
            printf("\r\nThe User is:card_4\r\n");
        }
        //total = card1_bit+card2_bit+card3_bit+card4_bit+card0_bit;
        status = PcdSelect(SN);//Ñ¡¿¨
    }
    else
    {
    }

    if(status == MI_OK)//Ñ¡¿¨³É¹¦
    {
        status = MI_ERR;
        // ÑéÖ¤AÃÜÔ¿ ¿éµØÖ· ÃÜÂë SN 
		// ×¢Òâ£º´Ë´¦µÄ¿éµØÖ·0x0B¼´2ÉÈÇø3Çø¿é£¬¿ÉÒÔÌæ»»³É±äÁ¿addr £¬´Ë¿éµØÖ·Ö»ÐèÒªÖ¸ÏòÄ³Ò»ÉÈÇø¾Í¿ÉÒÔÁË£¬¼´2ÉÈÇøÎª0x08-0x0BÕâ¸ö·¶Î§¶¼ÓÐÐ§£¬ÇÒÖ»ÄÜ¶ÔÑéÖ¤¹ýµÄÉÈÇø½øÐÐ¶ÁÐ´²Ù×÷
        status = PcdAuthState(0x60, addr, KEY_A, SN);
        if(status == MI_OK)//ÑéÖ¤³É¹¦
        {
            printf("PcdAuthState(A) success\r\n");
        }
        else
        {
            printf("PcdAuthState(A) failed\r\n");
        }
		// ÑéÖ¤BÃÜÔ¿ ¿éµØÖ· ÃÜÂë SN  ¿éµØÖ·0x0B¼´2ÉÈÇø3Çø¿é£¬¿ÉÒÔÌæ»»³É±äÁ¿addr
		status = PcdAuthState(0x61, addr, KEY_B, SN);
        if(status == MI_OK)//ÑéÖ¤³É¹¦
        {
            printf("PcdAuthState(B) success\r\n");
        }
        else
        {
            printf("PcdAuthState(B) failed\r\n");
        }
    }

    if(status == MI_OK)//ÑéÖ¤³É¹¦
    {
        status = MI_ERR;
        // ¶ÁÈ¡M1¿¨Ò»¿éÊý¾Ý ¿éµØÖ· ¶ÁÈ¡µÄÊý¾Ý ×¢Òâ£ºÒòÎªÉÏÃæÑéÖ¤µÄÉÈÇøÊÇ2ÉÈÇø£¬ËùÒÔÖ»ÄÜ¶Ô2ÉÈÇøµÄÊý¾Ý½øÐÐ¶ÁÐ´£¬¼´0x08-0x0BÕâ¸ö·¶Î§£¬³¬³ö·¶Î§¶ÁÈ¡Ê§°Ü¡£
        status = PcdRead(addr, DATA);  //¶ÔÉÈÇø¶ÁÈ¡
        if(status == MI_OK)//¶Á¿¨³É¹¦
        {
            // printf("RFID:%s\r\n", RFID);
            printf("DATA:");
            for(i = 0; i < 16; i++)
            {
                printf("%02x", DATA[i]);
            }
            printf("\r\n");
        }
        else
        {
            printf("PcdRead() failed\r\n");
        }
    }

    if(status == MI_OK)//¶Á¿¨³É¹¦
    {
        status = MI_ERR;
		printf("Write the card after 1 second. Do not move the card!!!\r\n");
		delay_ms(1000);
        // status = PcdWrite(addr, DATA0);
        // Ð´Êý¾Ýµ½M1¿¨Ò»¿é
        status = PcdWrite(addr, DATA1);
        if(status == MI_OK)//Ð´¿¨³É¹¦
        {
            printf("PcdWrite() success\r\n");
        }
        else
        {
            printf("PcdWrite() failed\r\n");
			delay_ms(3000);
        }
    }

    if(status == MI_OK)//Ð´¿¨³É¹¦
    {
        status = MI_ERR;
        // ¶ÁÈ¡M1¿¨Ò»¿éÊý¾Ý ¿éµØÖ· ¶ÁÈ¡µÄÊý¾Ý
        status = PcdRead(addr, DATA);
        if(status == MI_OK)//¶Á¿¨³É¹¦
        {
            // printf("DATA:%s\r\n", DATA);
            printf("DATA:");
            for(i = 0; i < 16; i++)
            {
                printf("%02x", DATA[i]);
            }
            printf("\r\n");
        }
        else
        {
            printf("PcdRead() failed\r\n");
        }
    }

    if(status == MI_OK)//¶Á¿¨³É¹¦
    {
        status = MI_ERR;
		printf("RC522_Handle() run finished after 1 second!\r\n");
        delay_ms(1000);
    }
}

//Ð´ÈëÉÈÇøÊý¾Ý  // sectionÉÈÇø
u8 RC522_write(u8 section,u8 * Sec_DATA)
{
    u8 i = 0;
    status = PcdRequest(PICC_REQALL,CT);//Ñ°¿¨

    // printf("\r\nstatus>>>>>>%d\r\n", status);

    if(status==MI_OK)// Ñ°¿¨³É¹¦
    {
        status=MI_ERR;
        status = PcdAnticoll(SN);// ·À³å×² »ñµÃUID ´æÈëSN
			
    }

    if (status==MI_OK)// ·À³å×²³É¹¦
    {
			
        status = MI_ERR;
        ShowID(SN); // ´®¿Ú´òÓ¡¿¨µÄIDºÅ UID
        status = PcdSelect(SN);//Ñ¡¿¨
    }
    else
    {
    }

    if(status == MI_OK)//Ñ¡¿¨³É¹¦
    {
        status = MI_ERR;
        // ÑéÖ¤AÃÜÔ¿ ¿éµØÖ· ÃÜÂë SN 
		// ×¢Òâ£º´Ë´¦µÄ¿éµØÖ·0x0B¼´2ÉÈÇø3Çø¿é£¬¿ÉÒÔÌæ»»³É±äÁ¿addr £¬´Ë¿éµØÖ·Ö»ÐèÒªÖ¸ÏòÄ³Ò»ÉÈÇø¾Í¿ÉÒÔÁË£¬¼´2ÉÈÇøÎª0x08-0x0BÕâ¸ö·¶Î§¶¼ÓÐÐ§£¬ÇÒÖ»ÄÜ¶ÔÑéÖ¤¹ýµÄÉÈÇø½øÐÐ¶ÁÐ´²Ù×÷
        status = PcdAuthState(0x60, section, KEY_A, SN);
        if(status == MI_OK)//ÑéÖ¤³É¹¦
        {
            printf("PcdAuthState(A) success\r\n");
        }
        else
        {
            printf("PcdAuthState(A) failed\r\n");
        }
		// ÑéÖ¤BÃÜÔ¿ ¿éµØÖ· ÃÜÂë SN  ¿éµØÖ·0x0B¼´2ÉÈÇø3Çø¿é£¬¿ÉÒÔÌæ»»³É±äÁ¿addr
		status = PcdAuthState(0x61, section, KEY_B, SN);
        if(status == MI_OK)//ÑéÖ¤³É¹¦
        {
            printf("PcdAuthState(B) success\r\n");
        }
        else
        {
            printf("PcdAuthState(B) failed\r\n");
        }
    }

    
    if(status == MI_OK)//¶Á¿¨³É¹¦
    {
        status = MI_ERR;
		printf("Write the card after 1 second. Do not move the card!!!\r\n");
		delay_ms(1000);
        // status = PcdWrite(addr, DATA0);
   			// Ð´Êý¾Ýµ½M1¿¨Ò»¿é
        status = PcdWrite(section,Sec_DATA);
        if(status == MI_OK)//Ð´¿¨³É¹¦
        {
            printf("PcdWrite() success\r\n");
        }
        else
        {
            printf("PcdWrite() failed\r\n");
			delay_ms(2000);
        }
    }

    if(status == MI_OK)//Ð´¿¨³É¹¦
    {
        status = MI_ERR;
        // ¶ÁÈ¡M1¿¨Ò»¿éÊý¾Ý ¿éµØÖ· ¶ÁÈ¡µÄÊý¾Ý
        status = PcdRead(section, DATA);
        if(status == MI_OK)//¶Á¿¨³É¹¦
        {
            // printf("DATA:%s\r\n", DATA);
            printf("DATA:");
            for(i = 0; i < 16; i++)
            {
                printf("%02x", DATA[i]);
            }
            printf("\r\n");
        }
        else
        {
            printf("PcdRead() failed\r\n");
        }
    }
		if(status == MI_OK)//Ð´¿¨³É¹¦
    {
			return 1;
		}
 	}


//¶ÁÈ¡Êý¾Ý
// sectionÉÈÇø  date ·µ»ØÊý¾Ý
void RC522_recognition(u8 section,u8 (*date)[16])
{
	  
    u8 i = 0;
    status = PcdRequest(PICC_REQALL,CT);//Ñ°¿¨

    // printf("\r\nstatus>>>>>>%d\r\n", status);

    if(status==MI_OK)// Ñ°¿¨³É¹¦
    {
        status=MI_ERR;
        status = PcdAnticoll(SN);// ·À³å×² »ñµÃUID ´æÈëSN
    }

    if (status==MI_OK)// ·À³å×²³É¹¦
    {
        status = MI_ERR;
        ShowID(SN); // ´®¿Ú´òÓ¡¿¨µÄIDºÅ UID

        
        status = PcdSelect(SN);//Ñ¡¿¨
    }
    else
    {
    }

    if(status == MI_OK)//Ñ¡¿¨³É¹¦
    {
        status = MI_ERR;
        // ÑéÖ¤AÃÜÔ¿ ¿éµØÖ· ÃÜÂë SN 
		// ×¢Òâ£º´Ë´¦µÄ¿éµØÖ·0x0B¼´2ÉÈÇø3Çø¿é£¬¿ÉÒÔÌæ»»³É±äÁ¿addr £¬´Ë¿éµØÖ·Ö»ÐèÒªÖ¸ÏòÄ³Ò»ÉÈÇø¾Í¿ÉÒÔÁË£¬¼´2ÉÈÇøÎª0x08-0x0BÕâ¸ö·¶Î§¶¼ÓÐÐ§£¬ÇÒÖ»ÄÜ¶ÔÑéÖ¤¹ýµÄÉÈÇø½øÐÐ¶ÁÐ´²Ù×÷
        status = PcdAuthState(0x60, section, KEY_A, SN);
        if(status == MI_OK)//ÑéÖ¤³É¹¦
        {
            printf("PcdAuthState(A) success\r\n");
        }
        else
        {
            printf("PcdAuthState(A) failed\r\n");
        }
		// ÑéÖ¤BÃÜÔ¿ ¿éµØÖ· ÃÜÂë SN  ¿éµØÖ·0x0B¼´2ÉÈÇø3Çø¿é£¬¿ÉÒÔÌæ»»³É±äÁ¿addr
		status = PcdAuthState(0x61, section, KEY_B, SN);
        if(status == MI_OK)//ÑéÖ¤³É¹¦
        {
            printf("PcdAuthState(B) success\r\n");
        }
        else
        {
            printf("PcdAuthState(B) failed\r\n");
        }
    }

    if(status == MI_OK)//ÑéÖ¤³É¹¦
    {
        status = MI_ERR;
        // ¶ÁÈ¡M1¿¨Ò»¿éÊý¾Ý ¿éµØÖ· ¶ÁÈ¡µÄÊý¾Ý ×¢Òâ£ºÒòÎªÉÏÃæÑéÖ¤µÄÉÈÇøÊÇ2ÉÈÇø£¬ËùÒÔÖ»ÄÜ¶Ô2ÉÈÇøµÄÊý¾Ý½øÐÐ¶ÁÐ´£¬¼´0x08-0x0BÕâ¸ö·¶Î§£¬³¬³ö·¶Î§¶ÁÈ¡Ê§°Ü¡£
        status = PcdRead(section, DATA);
        if(status == MI_OK)//¶Á¿¨³É¹¦
        {
           
            printf("DATA:");
            for(i = 0; i < 16; i++)
            {
							
              printf("%02x", DATA[i]);
            }
            printf("\r\n");
						memcpy(date, DATA, sizeof(DATA));
						
        }
        else
        {
            printf("PcdRead() failed\r\n");
        }
    }

    
}


// ²âÊÔ³ÌÐò1£¬Íê³É0x0F¿é ÑéÖ¤KEY_A¡¢KEY_B ¶Á Ð´RFID1 ÑéÖ¤KEY_A1¡¢KEY_B1 ¶Á Ð´RFID2
void RC522_Handle1(void)
{
    u8 i = 0;
	unsigned char test_addr=0x0F;
    status = PcdRequest(PICC_REQALL,CT);//Ñ°¿¨

    // printf("\r\nstatus>>>>>>%d\r\n", status);

    if(status==MI_OK)// Ñ°¿¨³É¹¦
    {
        status=MI_ERR;
        status = PcdAnticoll(SN);// ·À³å×² »ñµÃUID ´æÈëSN
    }

    if (status==MI_OK)// ·À³å×²³É¹¦
    {
        status = MI_ERR;
        ShowID(SN); // ´®¿Ú´òÓ¡¿¨µÄIDºÅ UID

        // ÄÑµÀ¾ÍÊÇÎªÁË×ö¸öÅÐ¶ÏÂð¡£¡£¡£
        if((SN[0]==card_0[0])&&(SN[1]==card_0[1])&&(SN[2]==card_0[2])&&(SN[3]==card_0[3]))
        {
            card0_bit=1;
            printf("\r\nThe User is:card_0\r\n");
        }
        if((SN[0]==card_1[0])&&(SN[1]==card_1[1])&&(SN[2]==card_1[2])&&(SN[3]==card_1[3]))
        {
            card1_bit=1;
            printf("\r\nThe User is:card_1\r\n");
        }
        
        status = PcdSelect(SN);
    }
    else
    {
    }

    if(status == MI_OK)//Ñ¡¿¨³É¹¦
    {
        status = MI_ERR;
        // ÑéÖ¤AÃÜÔ¿ ¿éµØÖ· ÃÜÂë SN 
		// ×¢Òâ£º´Ë´¦µÄ¿éµØÖ·0x0F¼´3ÉÈÇø3Çø¿é£¬´Ë¿éµØÖ·Ö»ÐèÒªÖ¸ÏòÄ³Ò»ÉÈÇø¾Í¿ÉÒÔÁË£¬¼´3ÉÈÇøÎª0x0C-0x0FÕâ¸ö·¶Î§¶¼ÓÐÐ§£¬ÇÒÖ»ÄÜ¶ÔÑéÖ¤¹ýµÄÉÈÇø½øÐÐ¶ÁÐ´²Ù×÷
        status = PcdAuthState(0x60, test_addr, KEY_A, SN);
        if(status == MI_OK)//ÑéÖ¤³É¹¦
        {
            printf("PcdAuthState(A) success\r\n");
        }
        else
        {
            printf("PcdAuthState(A) failed\r\n");
			status = MI_OK;
			goto P1;
        }
		// ÑéÖ¤BÃÜÔ¿ ¿éµØÖ· ÃÜÂë SN 
		status = PcdAuthState(0x61, test_addr, KEY_B, SN);
        if(status == MI_OK)//ÑéÖ¤³É¹¦
        {
            printf("PcdAuthState(B) success\r\n");
        }
        else
        {
            printf("PcdAuthState(B) failed\r\n");
        }
    }

    if(status == MI_OK)//ÑéÖ¤³É¹¦
    {
        status = MI_ERR;
        // ¶ÁÈ¡M1¿¨Ò»¿éÊý¾Ý ¿éµØÖ· ¶ÁÈ¡µÄÊý¾Ý ×¢Òâ£ºÒòÎªÉÏÃæÑéÖ¤µÄÉÈÇøÊÇ3ÉÈÇø£¬ËùÒÔÖ»ÄÜ¶Ô2ÉÈÇøµÄÊý¾Ý½øÐÐ¶ÁÐ´£¬¼´0x0C-0x0FÕâ¸ö·¶Î§£¬³¬³ö·¶Î§¶ÁÈ¡Ê§°Ü¡£
        status = PcdRead(test_addr, DATA);
        if(status == MI_OK)//¶Á¿¨³É¹¦
        {
            // printf("RFID:%s\r\n", RFID);
            printf("DATA:");
            for(i = 0; i < 16; i++)
            {
                printf("%02x", DATA[i]);
            }
            printf("\r\n");
        }
        else
        {
            printf("PcdRead() failed\r\n");
        }
    }

    if(status == MI_OK)//¶Á¿¨³É¹¦
    {
        status = MI_ERR;
        // Ð´Êý¾Ýµ½M1¿¨Ò»¿é
        status = PcdWrite(test_addr, RFID1);
        if(status == MI_OK)//Ð´¿¨³É¹¦
        {
            printf("PcdWrite(RFID1) success\r\n");
        }
        else
        {
            printf("PcdWrite(RFID1) failed\r\n");
			delay_ms(3000);
        }
    }

P1:	
	if(status == MI_OK)//Ð´¿¨³É¹¦
    {
        status = MI_ERR;
        // ÑéÖ¤AÃÜÔ¿ ¿éµØÖ· ÃÜÂë SN 
		// ×¢Òâ£º´Ë´¦µÄ¿éµØÖ·0x0F¼´3ÉÈÇø3Çø¿é£¬´Ë¿éµØÖ·Ö»ÐèÒªÖ¸ÏòÄ³Ò»ÉÈÇø¾Í¿ÉÒÔÁË£¬¼´3ÉÈÇøÎª0x0C-0x0FÕâ¸ö·¶Î§¶¼ÓÐÐ§£¬ÇÒÖ»ÄÜ¶ÔÑéÖ¤¹ýµÄÉÈÇø½øÐÐ¶ÁÐ´²Ù×÷
        status = PcdAuthState(0x60, test_addr, KEY_A1, SN);
        if(status == MI_OK)//ÑéÖ¤³É¹¦
        {
            printf("PcdAuthState(A1) success\r\n");
        }
        else
        {
            printf("PcdAuthState(A1) failed\r\n");
        }
		// ÑéÖ¤BÃÜÔ¿ ¿éµØÖ· ÃÜÂë SN 
		status = PcdAuthState(0x61, test_addr, KEY_B1, SN);
        if(status == MI_OK)//ÑéÖ¤³É¹¦
        {
            printf("PcdAuthState(B1) success\r\n");
        }
        else
        {
            printf("PcdAuthState(B1) failed\r\n");
        }
    }
	
	if(status == MI_OK)//ÑéÖ¤³É¹¦
    {
        status = MI_ERR;
        // ¶ÁÈ¡M1¿¨Ò»¿éÊý¾Ý ¿éµØÖ· ¶ÁÈ¡µÄÊý¾Ý ×¢Òâ£ºÒòÎªÉÏÃæÑéÖ¤µÄÉÈÇøÊÇ3ÉÈÇø£¬ËùÒÔÖ»ÄÜ¶Ô2ÉÈÇøµÄÊý¾Ý½øÐÐ¶ÁÐ´£¬¼´0x0C-0x0FÕâ¸ö·¶Î§£¬³¬³ö·¶Î§¶ÁÈ¡Ê§°Ü¡£
        status = PcdRead(test_addr, DATA);
        if(status == MI_OK)//¶Á¿¨³É¹¦
        {
            // printf("RFID:%s\r\n", RFID);
            printf("DATA:");
            for(i = 0; i < 16; i++)
            {
                printf("%02x", DATA[i]);
            }
            printf("\r\n");
        }
        else
        {
            printf("PcdRead() failed\r\n");
        }
    }
	
	if(status == MI_OK)//¶Á¿¨³É¹¦
    {
        status = MI_ERR;
        // Ð´Êý¾Ýµ½M1¿¨Ò»¿é
        status = PcdWrite(test_addr, RFID2);
        if(status == MI_OK)//Ð´¿¨³É¹¦
        {
            printf("PcdWrite(RFID2) success\r\n");
        }
        else
        {
            printf("PcdWrite(RFID2) failed\r\n");
			delay_ms(3000);
        }
    }

    if(status == MI_OK)//Ð´¿¨³É¹¦
    {
        status = MI_ERR;
        // ¶ÁÈ¡M1¿¨Ò»¿éÊý¾Ý ¿éµØÖ· ¶ÁÈ¡µÄÊý¾Ý
        status = PcdRead(test_addr, DATA);
        if(status == MI_OK)//¶Á¿¨³É¹¦
        {
            // printf("DATA:%s\r\n", DATA);
            printf("DATA:");
            for(i = 0; i < 16; i++)
            {
                printf("%02x", DATA[i]);
            }
            printf("\r\n");
        }
        else
        {
            printf("PcdRead() failed\r\n");
        }
    }

    if(status == MI_OK)//¶Á¿¨³É¹¦
    {
        status = MI_ERR;
		printf("RC522_Handle1() run finished after 1 second!\r\n");
        delay_ms(1000);
    }
}

// ²âÊÔÓÃÊý¾Ý±¬ÆÆ³ÌÐò£¬½ö¹©Ñ§Ï°²Î¿¼£¬ÇëÎð·Ç·¨Ê¹ÓÃ Õë¶Ôcard_0½øÐÐÆÆ½â
void RC522_data_break(void)
{
	// ±¬ÆÆµÄ¿éµØÖ·
	unsigned char break_addr = 0x0F;
	u8 i = 0;
	/*
	u8 key_arr[257] = {	0x00, 0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07, 0x08, 0x09, 0x0A, 0x0B, 0x0C, 0x0D, 0x0E, 0x0F, 
						0x10, 0x11, 0x12, 0x13, 0x14, 0x15, 0x16, 0x17, 0x18, 0x19, 0x1A, 0x1B, 0x1C, 0x1D, 0x1E, 0x1F, 
						0x20, 0x21, 0x22, 0x23, 0x24, 0x25, 0x26, 0x27, 0x28, 0x29, 0x2A, 0x2B, 0x2C, 0x2D, 0x2E, 0x2F, 
						0x30, 0x31, 0x32, 0x33, 0x34, 0x35, 0x36, 0x37, 0x38, 0x39, 0x3A, 0x3B, 0x3C, 0x3D, 0x3E, 0x3F, 
						0x40, 0x41, 0x42, 0x43, 0x44, 0x45, 0x46, 0x47, 0x48, 0x49, 0x4A, 0x4B, 0x4C, 0x4D, 0x4E, 0x4F, 
						0x50, 0x51, 0x52, 0x53, 0x54, 0x55, 0x56, 0x57, 0x58, 0x59, 0x5A, 0x5B, 0x5C, 0x5D, 0x5E, 0x5F, 
						0x60, 0x61, 0x62, 0x63, 0x64, 0x65, 0x66, 0x67, 0x68, 0x69, 0x6A, 0x6B, 0x6C, 0x6D, 0x6E, 0x6F, 
						0x70, 0x71, 0x72, 0x73, 0x74, 0x75, 0x76, 0x77, 0x78, 0x79, 0x7A, 0x7B, 0x7C, 0x7D, 0x7E, 0x7F, 
						0x80, 0x81, 0x82, 0x83, 0x84, 0x85, 0x86, 0x87, 0x88, 0x89, 0x8A, 0x8B, 0x8C, 0x8D, 0x8E, 0x8F, 
						0x90, 0x91, 0x92, 0x93, 0x94, 0x95, 0x96, 0x97, 0x98, 0x99, 0x9A, 0x9B, 0x9C, 0x9D, 0x9E, 0x9F, 
						0xA0, 0xA1, 0xA2, 0xA3, 0xA4, 0xA5, 0xA6, 0xA7, 0xA8, 0xA9, 0xAA, 0xAB, 0xAC, 0xAD, 0xAE, 0xAF, 
						0xB0, 0xB1, 0xB2, 0xB3, 0xB4, 0xB5, 0xB6, 0xB7, 0xB8, 0xB9, 0xBA, 0xBB, 0xBC, 0xBD, 0xBE, 0xBF, 
						0xC0, 0xC1, 0xC2, 0xC3, 0xC4, 0xC5, 0xC6, 0xC7, 0xC8, 0xC9, 0xCA, 0xCB, 0xCC, 0xCD, 0xCE, 0xCF, 
						0xD0, 0xD1, 0xD2, 0xD3, 0xD4, 0xD5, 0xD6, 0xD7, 0xD8, 0xD9, 0xDA, 0xDB, 0xDC, 0xDD, 0xDE, 0xDF, 
						0xE0, 0xE1, 0xE2, 0xE3, 0xE4, 0xE5, 0xE6, 0xE7, 0xE8, 0xE9, 0xEA, 0xEB, 0xEC, 0xED, 0xEE, 0xEF, 
						0xF0, 0xF1, 0xF2, 0xF3, 0xF4, 0xF5, 0xF6, 0xF7, 0xF8, 0xF9, 0xFA, 0xFB, 0xFC, 0xFD, 0xFE, 0xFF };
	*/
	u8 break_KEY[6]= {0, 0, 0, 0, 0, 0};
	
    status = PcdRequest(PICC_REQALL,CT);//Ñ°¿¨

    // printf("\r\nstatus>>>>>>%d\r\n", status);

    if(status==MI_OK)// Ñ°¿¨³É¹¦
    {
        status=MI_ERR;
        status = PcdAnticoll(SN);// ·À³å×² »ñµÃUID ´æÈëSN
    }

    if (status==MI_OK)// ·À³å×²³É¹¦
    {
        status = MI_ERR;
        ShowID(SN); // ´®¿Ú´òÓ¡¿¨µÄIDºÅ UID

        // ÄÑµÀ¾ÍÊÇÎªÁË×ö¸öÅÐ¶ÏÂð¡£¡£¡£
        if((SN[0]==card_0[0])&&(SN[1]==card_0[1])&&(SN[2]==card_0[2])&&(SN[3]==card_0[3]))
        {
            card0_bit=1;
            printf("\r\nThe User is:card_0\r\n");
        }
		else
		{
			printf("\r\nThe User isn't:card_0\r\n");
			return;
		}
        
        status = PcdSelect(SN);
    }
    else
    {
    }

    if(status == MI_OK)//Ñ¡¿¨³É¹¦
    {
        status = MI_ERR;
		
		// ×ÔÓÉ·¢»Ó ¡£¡£¡£
		
		// ÑéÖ¤AÃÜÔ¿ ¿éµØÖ· ÃÜÂë SN 
		// ×¢Òâ£º´Ë´¦µÄ¿éµØÖ·0x0F¼´3ÉÈÇø3Çø¿é£¬´Ë¿éµØÖ·Ö»ÐèÒªÖ¸ÏòÄ³Ò»ÉÈÇø¾Í¿ÉÒÔÁË£¬¼´3ÉÈÇøÎª0x0C-0x0FÕâ¸ö·¶Î§¶¼ÓÐÐ§£¬ÇÒÖ»ÄÜ¶ÔÑéÖ¤¹ýµÄÉÈÇø½øÐÐ¶ÁÐ´²Ù×÷
		status = PcdAuthState(0x60, break_addr, break_KEY, SN);
		if(status == MI_OK)//ÑéÖ¤³É¹¦
		{
			printf("PcdAuthState(A) success\r\n");
		}
		else
		{
			printf("PcdAuthState(A) failed\r\n");
			status = MI_OK;
		}
		
		// ÑéÖ¤BÃÜÔ¿ ¿éµØÖ· ÃÜÂë SN 
		status = PcdAuthState(0x61, break_addr, break_KEY, SN);
		if(status == MI_OK)//ÑéÖ¤³É¹¦
		{
			printf("PcdAuthState(B) success\r\n");
		}
		else
		{
			printf("PcdAuthState(B) failed\r\n");
		}
    }

    if(status == MI_OK)//ÑéÖ¤³É¹¦
    {
        status = MI_ERR;
        // ¶ÁÈ¡M1¿¨Ò»¿éÊý¾Ý ¿éµØÖ· ¶ÁÈ¡µÄÊý¾Ý ×¢Òâ£ºÒòÎªÉÏÃæÑéÖ¤µÄÉÈÇøÊÇ3ÉÈÇø£¬ËùÒÔÖ»ÄÜ¶Ô2ÉÈÇøµÄÊý¾Ý½øÐÐ¶ÁÐ´£¬¼´0x0C-0x0FÕâ¸ö·¶Î§£¬³¬³ö·¶Î§¶ÁÈ¡Ê§°Ü¡£
        status = PcdRead(break_addr, DATA);
        if(status == MI_OK)//¶Á¿¨³É¹¦
        {
            // printf("RFID:%s\r\n", RFID);
            printf("DATA:");
            for(i = 0; i < 16; i++)
            {
                printf("%02x", DATA[i]);
            }
            printf("\r\n");
        }
        else
        {
            printf("PcdRead() failed\r\n");
        }
    }

    delay_ms(3000);
}

void RC522_Init ( void )
{
    SPI1_Init();

    RC522_Reset_Disable();

    RC522_CS_Disable();

    PcdReset ();

    M500PcdConfigISOType ( 'A' );//ÉèÖÃ¹¤×÷·½Ê½

}

void SPI1_Init(void)
{
    GPIO_InitTypeDef GPIO_InitStructure;
    SPI_InitTypeDef  SPI_InitStructure;
    RCC_APB2PeriphClockCmd(	RCC_APB2Periph_GPIOA | RCC_APB2Periph_GPIOB, ENABLE );//PORTA¡¢BÊ±ÖÓÊ¹ÄÜ
    RCC_APB1PeriphClockCmd(	RCC_APB2Periph_SPI1,  ENABLE );												//SPI1Ê±ÖÓÊ¹ÄÜ

    // CS
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_4;
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP; 		 //ÍÆÍìÊä³ö
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;		 //IO¿ÚËÙ¶ÈÎª50MHz
    GPIO_Init(GPIOA, &GPIO_InitStructure);					 //¸ù¾ÝÉè¶¨²ÎÊý³õÊ¼»¯PF0¡¢PF1

    // SCK
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_5;
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP; 		 //ÍÆÍìÊä³ö
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;		 //IO¿ÚËÙ¶ÈÎª50MHz
    GPIO_Init(GPIOA, &GPIO_InitStructure);

    // MISO
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_6;
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IN_FLOATING; 		 //ÍÆÍìÊä³ö
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;		 //IO¿ÚËÙ¶ÈÎª50MHz
    GPIO_Init(GPIOA, &GPIO_InitStructure);

    // MOSI
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_7;
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP; 		 //ÍÆÍìÊä³ö
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;		 //IO¿ÚËÙ¶ÈÎª50MHz
    GPIO_Init(GPIOA, &GPIO_InitStructure);

    // RST
    GPIO_InitStructure.GPIO_Pin = GPIO_Pin_0;
    GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP; 		 //ÍÆÍìÊä³ö
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;		 //IO¿ÚËÙ¶ÈÎª50MHz
    GPIO_Init(GPIOB, &GPIO_InitStructure);

    SPI_InitStructure.SPI_Direction = SPI_Direction_2Lines_FullDuplex;  					//ÉèÖÃSPIµ¥Ïò»òÕßË«ÏòµÄÊý¾ÝÄ£Ê½:SPIÉèÖÃÎªË«ÏßË«ÏòÈ«Ë«¹¤
    SPI_InitStructure.SPI_Mode = SPI_Mode_Master;																	//ÉèÖÃSPI¹¤×÷Ä£Ê½:ÉèÖÃÎªÖ÷SPI
    SPI_InitStructure.SPI_DataSize = SPI_DataSize_8b;															//ÉèÖÃSPIµÄÊý¾Ý´óÐ¡:SPI·¢ËÍ½ÓÊÕ8Î»Ö¡½á¹¹
    SPI_InitStructure.SPI_CPOL = SPI_CPOL_High;																		//´®ÐÐÍ¬²½Ê±ÖÓµÄ¿ÕÏÐ×´Ì¬Îª¸ßµçÆ½
    // SPI_InitStructure.SPI_CPOL = SPI_CPOL_Low;
    // SPI_InitStructure.SPI_CPHA = SPI_CPHA_1Edge;																	//´®ÐÐÍ¬²½Ê±ÖÓµÄµÚÒ»¸öÌø±äÑØ£¨ÏÂ½µ£©Êý¾Ý±»²ÉÑù
    SPI_InitStructure.SPI_CPHA = SPI_CPHA_2Edge;																		//´®ÐÐÍ¬²½Ê±ÖÓµÄµÚ¶þ¸öÌø±äÑØ£¨ÉÏÉý£©Êý¾Ý±»²ÉÑù
    SPI_InitStructure.SPI_NSS = SPI_NSS_Soft;																			//NSSÐÅºÅÓÉÓ²¼þ£¨NSS¹Ü½Å£©»¹ÊÇÈí¼þ£¨Ê¹ÓÃSSIÎ»£©¹ÜÀí:ÄÚ²¿NSSÐÅºÅÓÐSSIÎ»¿ØÖÆ
    // RC522 SPIÍ¨Ñ¶Ê±ÖÓÖÜÆÚ×îÐ¡Îª100ns	¼´ÆµÂÊ×î´óÎª10MHZ
    // RC522 Êý¾ÝÔÚÏÂ½µÑØ±ä»¯
    SPI_InitStructure.SPI_BaudRatePrescaler = SPI_BaudRatePrescaler_256;					//¶¨Òå²¨ÌØÂÊÔ¤·ÖÆµµÄÖµ:²¨ÌØÂÊÔ¤·ÖÆµÖµÎª256¡¢´«ÊäËÙÂÊ36M/256=140.625KHz
    SPI_InitStructure.SPI_FirstBit = SPI_FirstBit_MSB;														//Ö¸¶¨Êý¾Ý´«Êä´ÓMSBÎ»»¹ÊÇLSBÎ»¿ªÊ¼:Êý¾Ý´«Êä´ÓMSBÎ»¿ªÊ¼
    SPI_InitStructure.SPI_CRCPolynomial = 7;																			//CRCÖµ¼ÆËãµÄ¶àÏîÊ½
    SPI_Init(SPI1, &SPI_InitStructure); 						 															//¸ù¾ÝSPI_InitStructÖÐÖ¸¶¨µÄ²ÎÊý³õÊ¼»¯ÍâÉèSPIx¼Ä´æÆ÷

    SPI_Cmd(SPI1, ENABLE); //Ê¹ÄÜSPIÍâÉè
}


/*
 * º¯ÊýÃû£ºSPI_RC522_SendByte
 * ÃèÊö  £ºÏòRC522·¢ËÍ1 Byte Êý¾Ý
 * ÊäÈë  £ºbyte£¬Òª·¢ËÍµÄÊý¾Ý
 * ·µ»Ø  : RC522·µ»ØµÄÊý¾Ý
 * µ÷ÓÃ  £ºÄÚ²¿µ÷ÓÃ
 */
void SPI_RC522_SendByte ( u8 byte )
{
    u8 counter;

    for(counter=0; counter<8; counter++)
    {
        if ( byte & 0x80 )
            RC522_MOSI_1 ();
        else
            RC522_MOSI_0 ();

        RC522_DELAY();
        RC522_SCK_0 ();
        RC522_DELAY();
        RC522_SCK_1();
        RC522_DELAY();

        byte <<= 1;
    }
}


/*
 * º¯ÊýÃû£ºSPI_RC522_ReadByte
 * ÃèÊö  £º´ÓRC522·¢ËÍ1 Byte Êý¾Ý
 * ÊäÈë  £ºÎÞ
 * ·µ»Ø  : RC522·µ»ØµÄÊý¾Ý
 * µ÷ÓÃ  £ºÄÚ²¿µ÷ÓÃ
 */
u8 SPI_RC522_ReadByte ( void )
{
    u8 counter;
    u8 SPI_Data;

    for(counter=0; counter<8; counter++)
    {
        SPI_Data <<= 1;

        RC522_SCK_0 ();

        RC522_DELAY();

        if ( RC522_MISO_GET() == 1)
            SPI_Data |= 0x01;

        RC522_DELAY();

        RC522_SCK_1 ();

        RC522_DELAY();
    }

//	printf("****%c****",SPI_Data);
    return SPI_Data;
}


/*
 * º¯ÊýÃû£ºReadRawRC
 * ÃèÊö  £º¶ÁRC522¼Ä´æÆ÷
 * ÊäÈë  £ºucAddress£¬¼Ä´æÆ÷µØÖ·
 * ·µ»Ø  : ¼Ä´æÆ÷µÄµ±Ç°Öµ
 * µ÷ÓÃ  £ºÄÚ²¿µ÷ÓÃ
 */
u8 ReadRawRC ( u8 ucAddress )
{
    u8 ucAddr, ucReturn;

    ucAddr = ( ( ucAddress << 1 ) & 0x7E ) | 0x80;

    RC522_CS_Enable();

    SPI_RC522_SendByte ( ucAddr );

    ucReturn = SPI_RC522_ReadByte ();

    RC522_CS_Disable();

    return ucReturn;
}


/*
 * º¯ÊýÃû£ºWriteRawRC
 * ÃèÊö  £ºÐ´RC522¼Ä´æÆ÷
 * ÊäÈë  £ºucAddress£¬¼Ä´æÆ÷µØÖ·
 *         ucValue£¬Ð´Èë¼Ä´æÆ÷µÄÖµ
 * ·µ»Ø  : ÎÞ
 * µ÷ÓÃ  £ºÄÚ²¿µ÷ÓÃ
 */
void WriteRawRC ( u8 ucAddress, u8 ucValue )
{
    u8 ucAddr;

    ucAddr = ( ucAddress << 1 ) & 0x7E;

    RC522_CS_Enable();

    SPI_RC522_SendByte ( ucAddr );

    SPI_RC522_SendByte ( ucValue );

    RC522_CS_Disable();
}


/*
 * º¯ÊýÃû£ºSetBitMask
 * ÃèÊö  £º¶ÔRC522¼Ä´æÆ÷ÖÃÎ»
 * ÊäÈë  £ºucReg£¬¼Ä´æÆ÷µØÖ·
 *         ucMask£¬ÖÃÎ»Öµ
 * ·µ»Ø  : ÎÞ
 * µ÷ÓÃ  £ºÄÚ²¿µ÷ÓÃ
 */
void SetBitMask ( u8 ucReg, u8 ucMask )
{
    u8 ucTemp;

    ucTemp = ReadRawRC ( ucReg );

    WriteRawRC ( ucReg, ucTemp | ucMask );         // set bit mask
}


/*
 * º¯ÊýÃû£ºClearBitMask
 * ÃèÊö  £º¶ÔRC522¼Ä´æÆ÷ÇåÎ»
 * ÊäÈë  £ºucReg£¬¼Ä´æÆ÷µØÖ·
 *         ucMask£¬ÇåÎ»Öµ
 * ·µ»Ø  : ÎÞ
 * µ÷ÓÃ  £ºÄÚ²¿µ÷ÓÃ
 */
void ClearBitMask ( u8 ucReg, u8 ucMask )
{
    u8 ucTemp;

    ucTemp = ReadRawRC ( ucReg );

    WriteRawRC ( ucReg, ucTemp & ( ~ ucMask) );  // clear bit mask
}


/*
 * º¯ÊýÃû£ºPcdAntennaOn
 * ÃèÊö  £º¿ªÆôÌìÏß
 * ÊäÈë  £ºÎÞ
 * ·µ»Ø  : ÎÞ
 * µ÷ÓÃ  £ºÄÚ²¿µ÷ÓÃ
 */
void PcdAntennaOn ( void )
{
    u8 uc;

    uc = ReadRawRC ( TxControlReg );

    if ( ! ( uc & 0x03 ) )
        SetBitMask(TxControlReg, 0x03);
}


/*
 * º¯ÊýÃû£ºPcdAntennaOff
 * ÃèÊö  £º¿ªÆôÌìÏß
 * ÊäÈë  £ºÎÞ
 * ·µ»Ø  : ÎÞ
 * µ÷ÓÃ  £ºÄÚ²¿µ÷ÓÃ
 */
void PcdAntennaOff ( void )
{
    ClearBitMask ( TxControlReg, 0x03 );
}


/*
 * º¯ÊýÃû£ºPcdRese
 * ÃèÊö  £º¸´Î»RC522
 * ÊäÈë  £ºÎÞ
 * ·µ»Ø  : ÎÞ
 * µ÷ÓÃ  £ºÍâ²¿µ÷ÓÃ
 */
void PcdReset ( void )
{
    RC522_Reset_Disable();

    delay_us ( 1 );

    RC522_Reset_Enable();

    delay_us ( 1 );

    RC522_Reset_Disable();

    delay_us ( 1 );

    WriteRawRC ( CommandReg, 0x0f );

    while ( ReadRawRC ( CommandReg ) & 0x10 );

    delay_us ( 1 );

    WriteRawRC ( ModeReg, 0x3D );            //¶¨Òå·¢ËÍºÍ½ÓÊÕ³£ÓÃÄ£Ê½ ºÍMifare¿¨Í¨Ñ¶£¬CRC³õÊ¼Öµ0x6363

    WriteRawRC ( TReloadRegL, 30 );          //16Î»¶¨Ê±Æ÷µÍÎ»
    WriteRawRC ( TReloadRegH, 0 );			 //16Î»¶¨Ê±Æ÷¸ßÎ»

    WriteRawRC ( TModeReg, 0x8D );		      //¶¨ÒåÄÚ²¿¶¨Ê±Æ÷µÄÉèÖÃ

    WriteRawRC ( TPrescalerReg, 0x3E );			 //ÉèÖÃ¶¨Ê±Æ÷·ÖÆµÏµÊý

    WriteRawRC ( TxAutoReg, 0x40 );				   //µ÷ÖÆ·¢ËÍÐÅºÅÎª100%ASK
}


/*
 * º¯ÊýÃû£ºM500PcdConfigISOType
 * ÃèÊö  £ºÉèÖÃRC522µÄ¹¤×÷·½Ê½
 * ÊäÈë  £ºucType£¬¹¤×÷·½Ê½
 * ·µ»Ø  : ÎÞ
 * µ÷ÓÃ  £ºÍâ²¿µ÷ÓÃ
 */
void M500PcdConfigISOType ( u8 ucType )
{
    if ( ucType == 'A')                     //ISO14443_A
    {
        ClearBitMask ( Status2Reg, 0x08 );

        WriteRawRC ( ModeReg, 0x3D );//3F

        WriteRawRC ( RxSelReg, 0x86 );//84

        WriteRawRC( RFCfgReg, 0x7F );   //4F

        WriteRawRC( TReloadRegL, 30 );//tmoLength);// TReloadVal = 'h6a =tmoLength(dec)

        WriteRawRC ( TReloadRegH, 0 );

        WriteRawRC ( TModeReg, 0x8D );

        WriteRawRC ( TPrescalerReg, 0x3E );

        delay_us ( 2 );

        PcdAntennaOn ();//¿ªÌìÏß
    }
}


/*
 * º¯ÊýÃû£ºPcdComMF522
 * ÃèÊö  £ºÍ¨¹ýRC522ºÍISO14443¿¨Í¨Ñ¶
 * ÊäÈë  £ºucCommand£¬RC522ÃüÁî×Ö
 *         pInData£¬Í¨¹ýRC522·¢ËÍµ½¿¨Æ¬µÄÊý¾Ý
 *         ucInLenByte£¬·¢ËÍÊý¾ÝµÄ×Ö½Ú³¤¶È
 *         pOutData£¬½ÓÊÕµ½µÄ¿¨Æ¬·µ»ØÊý¾Ý
 *         pOutLenBit£¬·µ»ØÊý¾ÝµÄÎ»³¤¶È
 * ·µ»Ø  : ×´Ì¬Öµ
 *         = MI_OK£¬³É¹¦
 * µ÷ÓÃ  £ºÄÚ²¿µ÷ÓÃ
 */
char PcdComMF522 ( u8 ucCommand, u8 * pInData, u8 ucInLenByte, u8 * pOutData, u32 * pOutLenBit )
{
    char cStatus = MI_ERR;
    u8 ucIrqEn   = 0x00;
    u8 ucWaitFor = 0x00;
    u8 ucLastBits;
    u8 ucN;
    u32 ul;

    switch ( ucCommand )
    {
    case PCD_AUTHENT:		//MifareÈÏÖ¤
        ucIrqEn   = 0x12;		//ÔÊÐí´íÎóÖÐ¶ÏÇëÇóErrIEn  ÔÊÐí¿ÕÏÐÖÐ¶ÏIdleIEn
        ucWaitFor = 0x10;		//ÈÏÖ¤Ñ°¿¨µÈ´ýÊ±ºò ²éÑ¯¿ÕÏÐÖÐ¶Ï±êÖ¾Î»
        break;

    case PCD_TRANSCEIVE:		//½ÓÊÕ·¢ËÍ ·¢ËÍ½ÓÊÕ
        ucIrqEn   = 0x77;		//ÔÊÐíTxIEn RxIEn IdleIEn LoAlertIEn ErrIEn TimerIEn
        ucWaitFor = 0x30;		//Ñ°¿¨µÈ´ýÊ±ºò ²éÑ¯½ÓÊÕÖÐ¶Ï±êÖ¾Î»Óë ¿ÕÏÐÖÐ¶Ï±êÖ¾Î»
        break;

    default:
        break;
    }

    WriteRawRC ( ComIEnReg, ucIrqEn | 0x80 );		//IRqInvÖÃÎ»¹Ü½ÅIRQÓëStatus1RegµÄIRqÎ»µÄÖµÏà·´
    ClearBitMask ( ComIrqReg, 0x80 );			//Set1¸ÃÎ»ÇåÁãÊ±£¬CommIRqRegµÄÆÁ±ÎÎ»ÇåÁã
    WriteRawRC ( CommandReg, PCD_IDLE );		//Ð´¿ÕÏÐÃüÁî
    SetBitMask ( FIFOLevelReg, 0x80 );			//ÖÃÎ»FlushBufferÇå³ýÄÚ²¿FIFOµÄ¶ÁºÍÐ´Ö¸ÕëÒÔ¼°ErrRegµÄBufferOvfl±êÖ¾Î»±»Çå³ý

    for ( ul = 0; ul < ucInLenByte; ul ++ )
        WriteRawRC ( FIFODataReg, pInData [ ul ] );    		//Ð´Êý¾Ý½øFIFOdata

    WriteRawRC ( CommandReg, ucCommand );					//Ð´ÃüÁî

    if ( ucCommand == PCD_TRANSCEIVE )
        SetBitMask(BitFramingReg,0x80);  				//StartSendÖÃÎ»Æô¶¯Êý¾Ý·¢ËÍ ¸ÃÎ»ÓëÊÕ·¢ÃüÁîÊ¹ÓÃÊ±²ÅÓÐÐ§

    ul = 1000;//¸ù¾ÝÊ±ÖÓÆµÂÊµ÷Õû£¬²Ù×÷M1¿¨×î´óµÈ´ýÊ±¼ä25ms

    do 														//ÈÏÖ¤ ÓëÑ°¿¨µÈ´ýÊ±¼ä
    {
        ucN = ReadRawRC ( ComIrqReg );							//²éÑ¯ÊÂ¼þÖÐ¶Ï
        ul --;
    } while ( ( ul != 0 ) && ( ! ( ucN & 0x01 ) ) && ( ! ( ucN & ucWaitFor ) ) );		//ÍË³öÌõ¼þi=0,¶¨Ê±Æ÷ÖÐ¶Ï£¬ÓëÐ´¿ÕÏÐÃüÁî

    ClearBitMask ( BitFramingReg, 0x80 );					//ÇåÀíÔÊÐíStartSendÎ»

    if ( ul != 0 )
    {
        if ( ! (( ReadRawRC ( ErrorReg ) & 0x1B )) )			//¶Á´íÎó±êÖ¾¼Ä´æÆ÷BufferOfI CollErr ParityErr ProtocolErr
        {
            cStatus = MI_OK;

            if ( ucN & ucIrqEn & 0x01 )					//ÊÇ·ñ·¢Éú¶¨Ê±Æ÷ÖÐ¶Ï
                cStatus = MI_NOTAGERR;

            if ( ucCommand == PCD_TRANSCEIVE )
            {
                ucN = ReadRawRC ( FIFOLevelReg );			//¶ÁFIFOÖÐ±£´æµÄ×Ö½ÚÊý

                ucLastBits = ReadRawRC ( ControlReg ) & 0x07;	//×îºó½ÓÊÕµ½µÃ×Ö½ÚµÄÓÐÐ§Î»Êý

                if ( ucLastBits )
                    * pOutLenBit = ( ucN - 1 ) * 8 + ucLastBits;   	//N¸ö×Ö½ÚÊý¼õÈ¥1£¨×îºóÒ»¸ö×Ö½Ú£©+×îºóÒ»Î»µÄÎ»Êý ¶ÁÈ¡µ½µÄÊý¾Ý×ÜÎ»Êý
                else
                    * pOutLenBit = ucN * 8;   					//×îºó½ÓÊÕµ½µÄ×Ö½ÚÕû¸ö×Ö½ÚÓÐÐ§

                if ( ucN == 0 )
                    ucN = 1;

                if ( ucN > MAXRLEN )
                    ucN = MAXRLEN;

                for ( ul = 0; ul < ucN; ul ++ )
                    pOutData [ ul ] = ReadRawRC ( FIFODataReg );
            }
        }
        else
            cStatus = MI_ERR;
//			printf(ErrorReg);
    }

    SetBitMask ( ControlReg, 0x80 );           // stop timer now
    WriteRawRC ( CommandReg, PCD_IDLE );

    return cStatus;
}


/*
 * º¯ÊýÃû£ºPcdRequest
 * ÃèÊö  £ºÑ°¿¨
 * ÊäÈë  £ºucReq_code£¬Ñ°¿¨·½Ê½
 *                     = 0x52£¬Ñ°¸ÐÓ¦ÇøÄÚËùÓÐ·ûºÏ14443A±ê×¼µÄ¿¨
 *                     = 0x26£¬Ñ°Î´½øÈëÐÝÃß×´Ì¬µÄ¿¨
 *         pTagType£¬¿¨Æ¬ÀàÐÍ´úÂë
 *                   = 0x4400£¬Mifare_UltraLight
 *                   = 0x0400£¬Mifare_One(S50)
 *                   = 0x0200£¬Mifare_One(S70)
 *                   = 0x0800£¬Mifare_Pro(X))
 *                   = 0x4403£¬Mifare_DESFire
 * ·µ»Ø  : ×´Ì¬Öµ
 *         = MI_OK£¬³É¹¦
 * µ÷ÓÃ  £ºÍâ²¿µ÷ÓÃ
 */
char PcdRequest ( u8 ucReq_code, u8 * pTagType )
{
    char cStatus;
    u8 ucComMF522Buf [ MAXRLEN ];
    u32 ulLen;

    ClearBitMask ( Status2Reg, 0x08 );	//ÇåÀíÖ¸Ê¾MIFARECyptolµ¥Ôª½ÓÍ¨ÒÔ¼°ËùÓÐ¿¨µÄÊý¾ÝÍ¨ÐÅ±»¼ÓÃÜµÄÇé¿ö
    WriteRawRC ( BitFramingReg, 0x07 );	//	·¢ËÍµÄ×îºóÒ»¸ö×Ö½ÚµÄ ÆßÎ»
    SetBitMask ( TxControlReg, 0x03 );	//TX1,TX2¹Ü½ÅµÄÊä³öÐÅºÅ´«µÝ¾­·¢ËÍµ÷ÖÆµÄ13.56µÄÄÜÁ¿ÔØ²¨ÐÅºÅ

    ucComMF522Buf [ 0 ] = ucReq_code;		//´æÈë ¿¨Æ¬ÃüÁî×Ö

    cStatus = PcdComMF522 ( PCD_TRANSCEIVE,	ucComMF522Buf, 1, ucComMF522Buf, & ulLen );	//Ñ°¿¨

    if ( ( cStatus == MI_OK ) && ( ulLen == 0x10 ) )	//Ñ°¿¨³É¹¦·µ»Ø¿¨ÀàÐÍ
    {
        * pTagType = ucComMF522Buf [ 0 ];
        * ( pTagType + 1 ) = ucComMF522Buf [ 1 ];
    }
    else
        cStatus = MI_ERR;

    return cStatus;
}


/*
 * º¯ÊýÃû£ºPcdAnticoll
 * ÃèÊö  £º·À³å×²
 * ÊäÈë  £ºpSnr£¬¿¨Æ¬ÐòÁÐºÅ£¬4×Ö½Ú
 * ·µ»Ø  : ×´Ì¬Öµ
 *         = MI_OK£¬³É¹¦
 * µ÷ÓÃ  £ºÍâ²¿µ÷ÓÃ
 */
char PcdAnticoll ( u8 * pSnr )
{
    char cStatus;
    u8 uc, ucSnr_check = 0;
    u8 ucComMF522Buf [ MAXRLEN ];
    u32 ulLen;

    ClearBitMask ( Status2Reg, 0x08 );		//ÇåMFCryptol OnÎ» Ö»ÓÐ³É¹¦Ö´ÐÐMFAuthentÃüÁîºó£¬¸ÃÎ»²ÅÄÜÖÃÎ»
    WriteRawRC ( BitFramingReg, 0x00);		//ÇåÀí¼Ä´æÆ÷ Í£Ö¹ÊÕ·¢
    ClearBitMask ( CollReg, 0x80 );			//ÇåValuesAfterCollËùÓÐ½ÓÊÕµÄÎ»ÔÚ³åÍ»ºó±»Çå³ý

    /*
    ²Î¿¼ISO14443Ð­Òé£ºhttps://blog.csdn.net/wowocpp/article/details/79910800
    PCD ·¢ËÍ SEL = ¡®93¡¯£¬NVB = ¡®20¡¯Á½¸ö×Ö½Ú
    ÆÈÊ¹ËùÓÐµÄÔÚ³¡µÄPICC·¢»ØÍêÕûµÄUID CLn×÷ÎªÓ¦´ð¡£
    */
    ucComMF522Buf [ 0 ] = 0x93;	//¿¨Æ¬·À³åÍ»ÃüÁî
    ucComMF522Buf [ 1 ] = 0x20;

    // ·¢ËÍ²¢½ÓÊÕÊý¾Ý ½ÓÊÕµÄÊý¾Ý´æ´¢ÓÚucComMF522Buf
    cStatus = PcdComMF522 ( PCD_TRANSCEIVE, ucComMF522Buf, 2, ucComMF522Buf, & ulLen);//Óë¿¨Æ¬Í¨ÐÅ

    if ( cStatus == MI_OK)		//Í¨ÐÅ³É¹¦
    {
        // ÊÕµ½µÄUID ´æÈëpSnr
        for ( uc = 0; uc < 4; uc ++ )
        {
            * ( pSnr + uc )  = ucComMF522Buf [ uc ];			//¶Á³öUID
            ucSnr_check ^= ucComMF522Buf [ uc ];
        }

        if ( ucSnr_check != ucComMF522Buf [ uc ] )
            cStatus = MI_ERR;

    }

    SetBitMask ( CollReg, 0x80 );

    return cStatus;
}


/*
 * º¯ÊýÃû£ºCalulateCRC
 * ÃèÊö  £ºÓÃRC522¼ÆËãCRC16
 * ÊäÈë  £ºpIndata£¬¼ÆËãCRC16µÄÊý×é
 *         ucLen£¬¼ÆËãCRC16µÄÊý×é×Ö½Ú³¤¶È
 *         pOutData£¬´æ·Å¼ÆËã½á¹û´æ·ÅµÄÊ×µØÖ·
 * ·µ»Ø  : ÎÞ
 * µ÷ÓÃ  £ºÄÚ²¿µ÷ÓÃ
 */
void CalulateCRC ( u8 * pIndata, u8 ucLen, u8 * pOutData )
{
    u8 uc, ucN;

    ClearBitMask(DivIrqReg, 0x04);

    WriteRawRC(CommandReg, PCD_IDLE);

    SetBitMask(FIFOLevelReg, 0x80);

    for ( uc = 0; uc < ucLen; uc ++)
        WriteRawRC ( FIFODataReg, * ( pIndata + uc ) );

    WriteRawRC ( CommandReg, PCD_CALCCRC );

    uc = 0xFF;

    do
    {
        ucN = ReadRawRC ( DivIrqReg );
        uc --;
    } while ( ( uc != 0 ) && ! ( ucN & 0x04 ) );

    pOutData [ 0 ] = ReadRawRC ( CRCResultRegL );
    pOutData [ 1 ] = ReadRawRC ( CRCResultRegM );
}


/*
 * º¯ÊýÃû£ºPcdSelect
 * ÃèÊö  £ºÑ¡¶¨¿¨Æ¬
 * ÊäÈë  £ºpSnr£¬¿¨Æ¬ÐòÁÐºÅ£¬4×Ö½Ú
 * ·µ»Ø  : ×´Ì¬Öµ
 *         = MI_OK£¬³É¹¦
 * µ÷ÓÃ  £ºÍâ²¿µ÷ÓÃ
 */
char PcdSelect ( u8 * pSnr )
{
    char cStatus;
    u8 uc;
    u8 ucComMF522Buf [ MAXRLEN ];
    u32  ulLen;

    // ·À³å×² 0x93
    ucComMF522Buf [ 0 ] = PICC_ANTICOLL1;
    // ¼ÙÉèÃ»ÓÐ³åÍ»£¬PCD Ö¸¶¨NVBÎª70£¬´ËÖµ±íÊ¾PCD½«·¢ËÍÍêÕûµÄUID CLn£¬Óë40Î»UID CLn Æ¥ÅäµÄPICC£¬ÒÔSAK×÷ÎªÓ¦´ð
    ucComMF522Buf [ 1 ] = 0x70;
    ucComMF522Buf [ 6 ] = 0;

    // 3 4 5 6Î»´æ·ÅUID£¬µÚ7Î»Ò»Ö±Òì»ò¡£¡£¡£
    for ( uc = 0; uc < 4; uc ++ )
    {
        ucComMF522Buf [ uc + 2 ] = * ( pSnr + uc );
        ucComMF522Buf [ 6 ] ^= * ( pSnr + uc );
    }

    // CRC(Ñ­»·ÈßÓàÐ£Ñé)
    CalulateCRC ( ucComMF522Buf, 7, & ucComMF522Buf [ 7 ] );

    ClearBitMask ( Status2Reg, 0x08 );

    // ·¢ËÍ²¢½ÓÊÕÊý¾Ý
    cStatus = PcdComMF522 ( PCD_TRANSCEIVE, ucComMF522Buf, 9, ucComMF522Buf, & ulLen );

    if ( ( cStatus == MI_OK ) && ( ulLen == 0x18 ) )
        cStatus = MI_OK;
    else
        cStatus = MI_ERR;

    return cStatus;
}


/*
 * º¯ÊýÃû£ºPcdAuthState
 * ÃèÊö  £ºÑéÖ¤¿¨Æ¬ÃÜÂë
 * ÊäÈë  £ºucAuth_mode£¬ÃÜÂëÑéÖ¤Ä£Ê½
 *                     = 0x60£¬ÑéÖ¤AÃÜÔ¿
 *                     = 0x61£¬ÑéÖ¤BÃÜÔ¿
 *         u8 ucAddr£¬¿éµØÖ·
 *         pKey£¬ÃÜÂë
 *         pSnr£¬¿¨Æ¬ÐòÁÐºÅ£¬4×Ö½Ú
 * ·µ»Ø  : ×´Ì¬Öµ
 *         = MI_OK£¬³É¹¦
 * µ÷ÓÃ  £ºÍâ²¿µ÷ÓÃ
 */
char PcdAuthState ( u8 ucAuth_mode, u8 ucAddr, u8 * pKey, u8 * pSnr )
{
    char cStatus;
    u8 uc, ucComMF522Buf [ MAXRLEN ];
    u32 ulLen;

    ucComMF522Buf [ 0 ] = ucAuth_mode;
    ucComMF522Buf [ 1 ] = ucAddr;

    for ( uc = 0; uc < 6; uc ++ )
        ucComMF522Buf [ uc + 2 ] = * ( pKey + uc );

    for ( uc = 0; uc < 6; uc ++ )
        ucComMF522Buf [ uc + 8 ] = * ( pSnr + uc );

    // printf("char PcdAuthState ( u8 ucAuth_mode, u8 ucAddr, u8 * pKey, u8 * pSnr )\r\n");
    // printf("before PcdComMF522() ucComMF522Buf:%s\r\n", ucComMF522Buf);

    // ÑéÖ¤ÃÜÔ¿ÃüÁî
    cStatus = PcdComMF522 ( PCD_AUTHENT, ucComMF522Buf, 12, ucComMF522Buf, & ulLen );

    // printf("after PcdComMF522() ucComMF522Buf:%s\r\n", ucComMF522Buf);

    if ( ( cStatus != MI_OK ) || ( ! ( ReadRawRC ( Status2Reg ) & 0x08 ) ) )
    {
//			if(cStatus != MI_OK)
//					printf("666")	;
//			else
//				printf("888");
        cStatus = MI_ERR;
    }

    return cStatus;
}


/*
 * º¯ÊýÃû£ºPcdWrite
 * ÃèÊö  £ºÐ´Êý¾Ýµ½M1¿¨Ò»¿é
 * ÊäÈë  £ºu8 ucAddr£¬¿éµØÖ·
 *         pData£¬Ð´ÈëµÄÊý¾Ý£¬16×Ö½Ú
 * ·µ»Ø  : ×´Ì¬Öµ
 *         = MI_OK£¬³É¹¦
 * µ÷ÓÃ  £ºÍâ²¿µ÷ÓÃ
 */
char PcdWrite ( u8 ucAddr, u8 * pData )
{
    char cStatus;
    u8 uc, ucComMF522Buf [ MAXRLEN ];
    u32 ulLen;

    ucComMF522Buf [ 0 ] = PICC_WRITE;
    ucComMF522Buf [ 1 ] = ucAddr;

    CalulateCRC ( ucComMF522Buf, 2, & ucComMF522Buf [ 2 ] );

    cStatus = PcdComMF522 ( PCD_TRANSCEIVE, ucComMF522Buf, 4, ucComMF522Buf, & ulLen );

    if ( ( cStatus != MI_OK ) || ( ulLen != 4 ) || ( ( ucComMF522Buf [ 0 ] & 0x0F ) != 0x0A ) )
        cStatus = MI_ERR;

    if ( cStatus == MI_OK )
    {
        memcpy(ucComMF522Buf, pData, 16);
        for ( uc = 0; uc < 16; uc ++ )
            ucComMF522Buf [ uc ] = * ( pData + uc );

        CalulateCRC ( ucComMF522Buf, 16, & ucComMF522Buf [ 16 ] );

        cStatus = PcdComMF522 ( PCD_TRANSCEIVE, ucComMF522Buf, 18, ucComMF522Buf, & ulLen );

        if ( ( cStatus != MI_OK ) || ( ulLen != 4 ) || ( ( ucComMF522Buf [ 0 ] & 0x0F ) != 0x0A ) )
            cStatus = MI_ERR;

    }
    return cStatus;
}


/*
 * º¯ÊýÃû£ºPcdRead
 * ÃèÊö  £º¶ÁÈ¡M1¿¨Ò»¿éÊý¾Ý
 * ÊäÈë  £ºu8 ucAddr£¬¿éµØÖ·
 *         pData£¬¶Á³öµÄÊý¾Ý£¬16×Ö½Ú
 * ·µ»Ø  : ×´Ì¬Öµ
 *         = MI_OK£¬³É¹¦
 * µ÷ÓÃ  £ºÍâ²¿µ÷ÓÃ
 */
char PcdRead ( u8 ucAddr, u8 * pData )
{
    char cStatus;
    u8 uc, ucComMF522Buf [ MAXRLEN ];
    u32 ulLen;

    ucComMF522Buf [ 0 ] = PICC_READ;
    ucComMF522Buf [ 1 ] = ucAddr;

    CalulateCRC ( ucComMF522Buf, 2, & ucComMF522Buf [ 2 ] );

    cStatus = PcdComMF522 ( PCD_TRANSCEIVE, ucComMF522Buf, 4, ucComMF522Buf, & ulLen );

    if ( ( cStatus == MI_OK ) && ( ulLen == 0x90 ) )
    {
        for ( uc = 0; uc < 16; uc ++ )
            * ( pData + uc ) = ucComMF522Buf [ uc ];
    }
    else
        cStatus = MI_ERR;

    return cStatus;
}


/*
 * º¯ÊýÃû£ºPcdHalt
 * ÃèÊö  £ºÃüÁî¿¨Æ¬½øÈëÐÝÃß×´Ì¬
 * ÊäÈë  £ºÎÞ
 * ·µ»Ø  : ×´Ì¬Öµ
 *         = MI_OK£¬³É¹¦
 * µ÷ÓÃ  £ºÍâ²¿µ÷ÓÃ
 */
char PcdHalt( void )
{
    u8 ucComMF522Buf [ MAXRLEN ];
    u32  ulLen;

    ucComMF522Buf [ 0 ] = PICC_HALT;
    ucComMF522Buf [ 1 ] = 0;

    CalulateCRC ( ucComMF522Buf, 2, & ucComMF522Buf [ 2 ] );
    PcdComMF522 ( PCD_TRANSCEIVE, ucComMF522Buf, 4, ucComMF522Buf, & ulLen );

    return MI_OK;
}


void IC_CMT ( u8 * UID, u8 * KEY, u8 RW, u8 * Dat )
{
    u8 ucArray_ID [ 4 ] = { 0 };//ÏÈºó´æ·ÅIC¿¨µÄÀàÐÍºÍUID(IC¿¨ÐòÁÐºÅ)

    PcdRequest ( 0x52, ucArray_ID );//Ñ°¿¨

    PcdAnticoll ( ucArray_ID );//·À³å×²

    PcdSelect ( UID );//Ñ¡¶¨¿¨

    PcdAuthState ( 0x60, 0x10, KEY, UID );//Ð£Ñé

    if ( RW )//¶ÁÐ´Ñ¡Ôñ£¬1ÊÇ¶Á£¬0ÊÇÐ´
        PcdRead ( 0x10, Dat );
    else
        PcdWrite ( 0x10, Dat );

    PcdHalt ();
}

// ÏÔÊ¾¿¨µÄ¿¨ºÅ£¬ÒÔÊ®Áù½øÖÆÏÔÊ¾
void ShowID(u8 *p)
{
    u8 num[9];
    u8 i;

    for(i=0; i<4; i++)
    {
        num[i*2] = p[i] / 16;
        num[i*2] > 9 ? (num[i*2] += '7') : (num[i*2] += '0');
        num[i*2+1] = p[i] % 16;
        num[i*2+1] > 9 ? (num[i*2+1] += '7') : (num[i*2+1] += '0');
    }
    num[8] = 0;
    printf("ID>>>%s\r\n", num);
}


