#pragma config FOSC = HS
#pragma config WDTE = OFF
#pragma config PWRTE = OFF
#pragma config BOREN = ON
#pragma config LVP = OFF
#pragma config CPD = OFF
#pragma config WRT = OFF
#pragma config CP = OFF

#include <xc.h>
#include <pic16f877a.h>

#define _XTAL_FREQ 4000000
#define SO_MAU 10
#define Baud_rate 9600
#define LCD_RS RB3
#define LCD_RW RB2
#define LCD_EN RB1
#define LCD_DATA PORTD
#define LCD_DIR_RS TRISB3
#define LCD_DIR_RW TRISB2
#define LCD_DIR_EN TRISB1
#define LCD_DIR_DATA TRISD
#define LCD_STROBE() ((LCD_EN = 1), NOP(), (LCD_EN=0))

void lcd_write(unsigned char c){        //Viết ký tự lên LCD
  __delay_us(70);
  LCD_DATA = c;
  LCD_STROBE();
}

void lcd_clear(void){                  //Xóa màn hình
  LCD_RS = 0;
  lcd_write(0x1);
  __delay_ms(5);
}

void lcd_puts(const char * s){        // cho phép gửi chuỗi kí tự hiện thị
  LCD_RS = 1;
  while(*s)
  lcd_write(*s++);
}

void lcd_putch(char c){           // cho phép gửi kí tự hiện thị
  LCD_RS = 1;
  lcd_write( c );
}

void lcd_goto(unsigned char pos){ //Con trỏ địa chỉ
  LCD_RS = 0;
  lcd_write(0x80+pos);
}

void lcd_init(){                  //Thiết lập Lcd
  LCD_DIR_RS = 0;
  LCD_DIR_RW = 0;
  LCD_DIR_EN = 0;
  LCD_DIR_DATA = 0x00;
  LCD_RS = 0;
  LCD_EN = 0;
  LCD_RW = 0;
  LCD_DATA = 0x00;
  __delay_ms(400);
  lcd_write(0x3C); // Function set;
  __delay_ms(10);
  lcd_write(0x0F); // Turn on display
  __delay_ms(10);
  lcd_write(0x01); // Clear display
  __delay_ms(10);
  lcd_write(0x02); // Go to home
  __delay_ms(10);
  lcd_write(0x06); // Entry mode set
  __delay_ms(10);
}
void Initialize_UART(void){
  TRISC6 = 1;
  TRISC7 = 1;
  SPBRG = ((_XTAL_FREQ/16)/Baud_rate) - 1;
  BRGH = 1;
  SYNC = 0; //che do khong dong ho
  SPEN = 1; // Enable serial TX, RX
  TXEN = 1; // enable transmission, bat che do truyen
  CREN = 1; // enable reception, cho phep nhan lien tuc
  TX9 = 0; // 8-bit reception selected
  RX9 = 0; // 8-bit reception mode selected
  RCIE=1;
  TXIE=1;
}
void UART_send_char(char a){
  while(!TXIF);
  TXREG = a;
}
void UART_send_string(char* a){
  while(*a)
  UART_send_char(*a++);
}

unsigned long time [SO_MAU];
unsigned long time_tmp = 0x00;
unsigned char index = 0x00;

void timer0_Init(){
  TMR0 = 0x00; // Xoa gia tri thanh ghi TMR0
  TMR0IE = 0; // Khong cho phep ngat Timer0
  OPTION_REG = 0xC7; // Thiet lap thanh ghi Option.
  // Hoạt động chế độ bộ đếm
  // Bộ chia tần số timer0, 1:256
  TMR0IE = 1; // Cho phep ngat Timer0
  GIE = 1; // Cho phep ngat toan cuc
}
void __interrupt () timer0 (void){
/* Ngat ngoai INT0 */
  if((INTF == 1) && (INTE == 1)){
    time[index] = time_tmp + TMR0;
    index++;
    if(index == SO_MAU)
    index = 0x00;
    TMR0 = 0x00;
    time_tmp = 0x00;
    INTF = 0;
  }
/* Ngat Timer0 */
  if((TMR0IE == 1) && (TMR0IF == 1)){
    time_tmp += 0xFF;
    TMR0 = 0x00;
    TMR0IF = 0;
  }
}
void main(void){
  TRISB4=0;
  unsigned int sc=1;
  unsigned char i;
  unsigned int tmp;
  float sum;
  Initialize_UART();
  lcd_init(); // Khoi tao LCD
  timer0_Init(); // Thiet lap Timer0
  lcd_clear(); // Xóa màn hình
  lcd_goto(0); // Ve dau dong
  lcd_puts("NHIP TIM:");
  lcd_goto(0x40); // Xuong dong thu 2
  lcd_puts("TT: ");
  UART_send_string("Nhip tim:");
  TXREG = 13;
  INTE = 1; // Cho phep ngat ngoai
  GIE = 1; // Cho phep ngat toan cuc
  for(i = 0; i < SO_MAU; i++){ // Xoa du lieu
    time[i] = 0x00;
    }
  while(1){
    sum = 0x00;
    for(i = 0; i <SO_MAU; i++){
      sum += time[i];
    }
    // Gia tri thoi gian trung binh thời gian của 1 nhip tim
    sum = sum/SO_MAU; 
    sum = 15625.0/(4*sum);
    sum = 60*sum; // Nhip tim trong 1 phut
    tmp =(unsigned int)sum;
    /*Hien thi LCD và UART*/
    lcd_goto(0x09);
    lcd_putch((tmp/100) + 48);
    UART_send_char((tmp/100) + 48);
    tmp = tmp%100;
    lcd_putch((tmp/10) + 48);
    UART_send_char((tmp/10) + 48);
    lcd_putch((tmp%10) + 48);
    UART_send_char((tmp%10) + 48);
    TXREG = 13;
    LCD_RS = 0;
    __delay_ms(600);
    /*Hien thi on dinh*/
    lcd_goto(0x43);
    if(tmp>160||tmp<70){
      lcd_puts("Khong on dinh");
      RB4=1;
    }else{
      lcd_puts ("Da on dinh");
      RB4=0;
    }
  }
}
