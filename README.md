# Cardless-Cash
ATM Transactions within a minute without the use of ATM Card.
using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Data;
using System.Drawing;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Windows.Forms;
using System.IO;
using System.IO.Ports;
using System.Data.SqlClient;
using System.Threading;

namespace WindowsFormsApplication3
{
    public partial class Form7: Form
    {

        SqlConnection cn;
        SqlCommand cm;
        public SerialPort smsport;
        public string sread, swrite;

        public Form7()
        {
            InitializeComponent();
        }

        private void timer1_Tick(object sender, EventArgs e)
        {

        }

        private void button1_Click(object sender, EventArgs e)
        {
            try
            {
                String write;
                if (smsport.IsOpen)
                {
                    smsport.Close();
                }
           
                smsport.Open();
                write = "";
                write = "AT+CMGS=" + '"' + "+917846972361" + '"' + "\r";
                Thread.Sleep(500);
                smsport.WriteLine(write);
                smsport.WriteTimeout = 10000;
                smsport.ReadTimeout = 100;

                write = "";
                write = "Server Has been Started " + "\u001A";
                Thread.Sleep(500);
                smsport.WriteLine(write);
                smsport.WriteTimeout = 10000;
                smsport.ReadTimeout = 100;
                MessageBox.Show("SERVER IS READY");
                timer1.Start();
            }
            catch (Exception ex)
            {
                MessageBox.Show(ex.Message);
            }
            finally
            {
                smsport.Close();
            }

        }

       
           
        private void smscheck(SerialPort smsport)
        {
            try
            {
                for (; ; )
                {
                    Thread.Sleep(1000);
                    if (smsport.IsOpen)
                    {
                        smsport.Close();
                    }
                    smsport.Open();
                    swrite = "";
                    swrite = "AT+CPIN?\r\n";
                    smsport.WriteLine(swrite);
                    smsport.ReadTimeout = 100;
                    Thread.Sleep(1000);
                    sread = smsport.ReadExisting();
                 
                    if (sread.EndsWith("+CPIN: READY\r\n\r\nOK\r\n"))
                    {
                        bool bsim = true;
                        if (bsim == true)
                        {
                          
                            smstext(smsport);
                            break;
                        }
                    }
                }
            }
            catch (Exception ex)
            {
                Console.WriteLine(ex);
            }
        }
        private void smstext(SerialPort smsport)
        {
            string read, write;
            try
            {
                Thread.Sleep(1000);
                if (smsport.IsOpen)
                {
                    smsport.Close();
                }                                                                                                                                     
                for (; ; )
                {
                    smsport.Open();
                    write = "";
                    write = "AT+CMGF=1\r\n";
                    smsport.WriteLine(write);
                    smsport.ReadTimeout = 100;
                    Thread.Sleep(1000);
                    read = smsport.ReadExisting();
                    if (read.EndsWith("OK\r\n"))
                    {
                     
                        smsreg(smsport);
                        break;
                    }
                }
            }
            catch (Exception ex)
            {
                Console.WriteLine(ex);
            }
        }
        private void smsreg(SerialPort smsport)
        {
            string read, write;
            try
            {
                for (; ; )
                {
                    if (smsport.IsOpen)
                    {
                        smsport.Close();
                    }
                    smsport.Open();
                    write = "";
                    write = "AT+CREG?\r\n";
                    smsport.WriteLine(write);
                    Thread.Sleep(1000);
                    smsport.ReadTimeout = 100;
                    read = smsport.ReadExisting();
                  
                    if (read.EndsWith("+CREG: 0,5\r\n\r\nOK\r\n") || read.EndsWith("+CREG: 0,1\r\n\r\nOK\r\n"))
                    {
                    
                        de1(smsport);
                        break;
                    }
                }
            }
            catch (Exception ex)
            {
                MessageBox.Show(ex.Message);
            }
        }
        private void de1(SerialPort smsport)
        {
            string read, write;
            try
            {
                if (smsport.IsOpen)
                {
                    smsport.Close();
                }
                smsport.Open();
                for (; ; )
                {
                    write = "";
                    write = "AT+CMGDA=" + '"' + "DEL ALL" + '"' + "\r\n";
                    smsport.WriteLine(write);
                    Thread.Sleep(10000);
                    smsport.ReadTimeout = 100;
                    read = "";
              
                    read = smsport.ReadExisting();
                    if (read.EndsWith("OK\r\n"))
                    {
                        MessageBox.Show("DELETE ALL MESSAGES READY TO READ");
                        break;
                    }
                }
            }
            catch (Exception ex)
            {
                MessageBox.Show(ex.Message);
            }
        }
        private void dbconnect()
        {
            cn = new SqlConnection("Data Source=(LocalDB)\\v11.0;AttachDbFilename=C:\\CARD\\WindowsFormsApplication3\\WindowsFormsApplication3\\cardless.mdf;Integrated Security=True;Connect Timeout=30");
            cn.Open();
        }
        private void cardsearch()
        {
            dbconnect();
            string sql;
         
            sql = "SELECT * FROM acholder where acnum = " + Convert.ToUInt64(textBox1.Text);
            cm = new SqlCommand(sql, cn);
            SqlDataReader rs = cm.ExecuteReader();
            if (rs.HasRows)
            {
                rs.Read();
                textBox2.Text = "valid";
                textBox3.Text = rs.GetValue(10).ToString();
                textBox4.Text = rs.GetValue(16).ToString();
            }
            else
            {
                textBox2.Text = " invalid";
                textBox3.Text = "+917846972361";
            }
            cn.Close();
        }

        private void sim_index3(SerialPort smsport)
        {

            try
            {
                if (smsport.IsOpen)
                {
                    smsport.Close();
                }

                smsport.Open();
                swrite = "AT+CMGR=3\r\n";

                smsport.ReadTimeout = 100;
                Thread.Sleep(1000);
                smsport.Write(swrite);
                Thread.Sleep(1000);
                smsport.WriteTimeout = 1000;
               sread = "";
                sread = smsport.ReadExisting();
                if (!sread.EndsWith("ERROR\r\n"))
                {
                    if (sread.Contains("ACNO"))
                    {
                        int v = sread.IndexOf("ACNO");
                        string v1 = sread.Substring(v, 12);
                        string[] v2 = v1.Split('O');
                        textBox1.Text = v1.Substring(4, 08);
                        cardsearch();
                        delete_index3(smsport);
                    }
                 
                }
            }
            catch (Exception ex)
            {
                MessageBox.Show(ex.Message);
            }
        }




        private void delete_index3(SerialPort smsport)
        {
            try
            {
                if (smsport.IsOpen)
                {
                    smsport.Close();
                }
                smsport.Open();
                Thread.Sleep(1000);
                swrite = "AT+CMGD=3\r\n";
                smsport.ReadTimeout = 1000;
                Thread.Sleep(1000);
                smsport.Write(swrite);
                smsport.WriteTimeout = 1000;
                sread = "";
                Thread.Sleep(1000);
                sread = smsport.ReadExisting();
                if (sread.EndsWith("OK\r\n"))
                {

                }

            }
            catch (Exception ex)
            {
                MessageBox.Show(ex.Message);
            }
            finally
            {
                smsport.Close();
            }
        }




         private void sim_index2(SerialPort smsport)
        {

            try
            {
                if (smsport.IsOpen)
                {
                    smsport.Close();
                }

                smsport.Open();
                swrite = "AT+CMGR=2\r\n";

                smsport.ReadTimeout = 100;
                Thread.Sleep(1000);
                smsport.Write(swrite);
                Thread.Sleep(1000);
                smsport.WriteTimeout = 1000;
                sread = "";
                sread = smsport.ReadExisting();
                if (!sread.EndsWith("ERROR\r\n"))
                {
                    if (sread.Contains("ACNO"))
                    {
                        int v = sread.IndexOf("ACNO");
                        string v1 = sread.Substring(v, 12);
                        string[] v2 = v1.Split('O');
                        textBox1.Text = v1.Substring(4, 08);
                        cardsearch();
                       

                        delete_index3(smsport);
                    }
                }
            }
            catch (Exception ex)
            {
                MessageBox.Show(ex.Message);
            }
        }

        private void delete_index2(SerialPort smsport)
        {
            try
            {
                if (smsport.IsOpen)
                {
                    smsport.Close();
                }
                smsport.Open();
                Thread.Sleep(1000);
                swrite = "AT+CMGD=2\r\n";
                smsport.ReadTimeout = 1000;
                Thread.Sleep(1000);
                smsport.Write(swrite);
                smsport.WriteTimeout = 1000;
                sread = "";
                Thread.Sleep(1000);
                sread = smsport.ReadExisting();
                if (sread.EndsWith("OK\r\n"))
                {

                }

            }
            catch (Exception ex)
            {
                MessageBox.Show(ex.Message);
            }
            finally
            {
                smsport.Close();
            }
        }

        private void sim_index1(SerialPort smsport)
        {

            try
            {
                if (smsport.IsOpen)
                {
                    smsport.Close();
                }

                smsport.Open();
                swrite = "AT+CMGR=1\r\n";
            
                smsport.ReadTimeout = 100;
                Thread.Sleep(1000);
                smsport.Write(swrite);
                Thread.Sleep(1000);
                smsport.WriteTimeout = 1000;
                sread = "";
                sread = smsport.ReadExisting();
            
                int v = sread.IndexOf("CARD");
            
                string v1 = sread.Substring(v, 12);
            
                string[] V2 = v1.Split('D');

                textBox1.Text += V2[1] + Environment.NewLine;
       
                if (!sread.EndsWith("ERROR\r\n"))
                {
                    if (sread.Contains("CARD"))
                    {
                      
                         v = sread.IndexOf("CARD");
                    
                       v1 = sread.Substring(v, 12);
                      
                        string[] v2 = v1.Split('D');
                        textBox1.Text = v1.Substring(4, 8);

                        cardsearch();

                        delete_index3(smsport);
                    }
                    
                }
            }
            catch (Exception ex)
            {
             //  MessageBox.Show(ex.Message);
            }
        }

        private void delete_index1(SerialPort smsport)
        {
            try
            {
                if (smsport.IsOpen)
                {
                    smsport.Close();
                }
                smsport.Open();
                Thread.Sleep(1000);
                swrite = "AT+CMGD=1\r\n";
                smsport.ReadTimeout = 1000;
                Thread.Sleep(1000);
                smsport.Write(swrite);
                smsport.WriteTimeout = 1000;
                sread = "";
                Thread.Sleep(1000);
                sread = smsport.ReadExisting();
                if (sread.EndsWith("OK\r\n"))
                {

                }

            }
            catch (Exception ex)
            {
                MessageBox.Show(ex.Message);
            }
            finally
            {
                smsport.Close();
            }
        }

        

        
        
        
        
        
        
        

        
            

        private void textBox1_TextChanged(object sender, EventArgs e)
        {
        }

        private void button2_Click(object sender, EventArgs e)
        {
       
        }

        private void textBox4_TextChanged(object sender, EventArgs e)
        {

            try
            {
                string write;
                if (smsport.IsOpen)
                {
                    smsport.Close();
                }
                smsport.Open();
                write = "";
                write = "AT+CMGS=" + '"'+"+91" + textBox3.Text + '"' + "\r";
                Thread.Sleep(500);
                smsport.WriteLine(write);
                smsport.WriteTimeout = 1000;
                smsport.ReadTimeout = 100;

                write = "";
                write = textBox4.Text + "\u001A";
                Thread.Sleep(500);
                smsport.WriteLine(write);
                smsport.WriteTimeout = 10000;
                smsport.ReadTimeout = 100;
                write = "";
                write = "^z";
                Thread.Sleep(500);
                smsport.WriteLine(write);
                smsport.WriteTimeout = 1000;
                smsport.ReadTimeout = 100;
                timer1.Start();
            }
            catch (Exception ex)
            {
                MessageBox.Show(ex.Message);
            }
            finally
            {
                smsport.Close();
            }
        }

        private void textBox2_TextChanged(object sender, EventArgs e)
        {

        }

        private void timer1_Tick_1(object sender, EventArgs e)
        {

            sim_index1(smsport);
            sim_index2(smsport);
            sim_index3(smsport);

        }
        private void passdisp()
        {
            dbconnect();

        }
        private void comboBox1_SelectedIndexChanged_1(object sender, EventArgs e)
        {
            string s = comboBox1.Text;
            smsport = new SerialPort(s);
            try
            {
                for (; ; )
                {
                    Thread.Sleep(1000);
                    if (smsport.IsOpen)
                    {
                        smsport.Close();
                    }
                    smsport.Open();
                    swrite = "";
                    swrite = "AT\r\n";
                    smsport.WriteLine(swrite);
                    smsport.WriteTimeout = 100;
                    smsport.ReadTimeout = 100;
                    Thread.Sleep(1000);
                
                    sread = smsport.ReadExisting();
                    if (sread.EndsWith("OK\r\n"))
                    {
                        bool modem = true;
                        sread = "";
                   
                        if (modem == true)
                        {
                            smscheck(smsport);
                            break;
                        }
                        Console.WriteLine("Modem Working...");
                    }
                }
            }
            catch (Exception ex)
            {
                Console.WriteLine("some.." + ex);
            }

        }

        private void Form7_Load(object sender, EventArgs e)
        {

        }

        private void button2_Click_1(object sender, EventArgs e)
        {
            Form f = new MDIParent1();
            f.Show();
            this.Close();
        }

        private void textBox4_TextChanged_1(object sender, EventArgs e)
        {
            try
            {
                string write;
                if (smsport.IsOpen)
                {
                    smsport.Close();
                }
                smsport.Open();
                write = "";
                write = "AT+CMGS=" + '"' + "+91" + textBox3.Text + '"' + "\r";
                Thread.Sleep(500);
                smsport.WriteLine(write);
                smsport.WriteTimeout = 10000;
                smsport.ReadTimeout = 100;
                write = "";
                write = textBox4.Text + "\u001A";
                Thread.Sleep(500);
                smsport.WriteLine(write);
                smsport.WriteTimeout = 10000;
                smsport.ReadTimeout = 100;
                write = "";
                write = "^z";
                Thread.Sleep(500);
                smsport.WriteLine(write);
                smsport.WriteTimeout = 10000;
                smsport.ReadTimeout = 100;

            }
            catch (Exception ex)
            {
                MessageBox.Show(ex.ToString());
            }

        }

        private void label2_Click(object sender, EventArgs e)
        {

        }
        }
    }
