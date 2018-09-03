# 1、demo的功能描述  
客户端通过一个中转服务器向另一个服务端进行数据交互，中转服务器监听某个端口，当客户端有socket连接该端口时，中转服务器启动一个线程，该线程负责将客户端发来的数据原样发送给服务端，并将服务端发来的回应信息原样发送给客户端。  
# 2、所遇的坑  
BufferedReader的read()和readLine()方法都为阻塞的，即在流传输数据到最后，readLine()方法会一直阻塞在读取数据状态，而不会返回null，其返回null的情况是在读取时发生异常（如连接中断等），同样，read()方法在读到最后也会阻塞。  
<font color="red">注意：在java的api文档中提到，在读到流的末尾read()方法会返回-1，readLine()方法会返回null，但实际上如何判断一个流到达末尾了呢？socket提供了半关闭流的概念，socket有 ***shutdownOutput()*** 和 ***shutdownInput()*** 两个方法，可以告知输出或输入已经结束，即到了流的末尾。而一般更常用的做法是约定一个结束符，在读到该结束符时则表示输入或输出已经结束。</font>  
本文的demo约定在发送数据时以在数据最后添加一行内容为"==END=="来标志数据发送结束  
# 3、客户端  
	package socket.pretest;
	
	import org.springframework.boot.ApplicationArguments;
	import org.springframework.boot.ApplicationRunner;
	import org.springframework.stereotype.Component;
	
	import java.io.*;
	import java.net.Socket;
	
	@Component
	//springboot工程启动即运行
	public class TestApplicationRunner implements ApplicationRunner{
    @Override
    public void run(ApplicationArguments args) throws Exception {
        //开启两个线程，分别发送数据
		new Thread(new ClientThread()).start();
        new Thread(new ClientThread()).start();
    }
	}
	
	class ClientThread implements Runnable{
	    @Override
	    public void run(){
	        try{
	            Socket socket = new Socket("localhost", 3574);//创建一个客户端连接
	            PrintStream ps = new PrintStream(socket.getOutputStream());
				//末尾增加一行"==END=="来标志结束
	            ps.println("我是来自客户端的数据:"+'\n'+Thread.currentThread().getName()+'\n'+"==END==");
	            BufferedReader br = new BufferedReader(new InputStreamReader(socket.getInputStream()));
	
	            String line = null;
	            StringBuffer sb = new StringBuffer("");
	            while (!(line = br.readLine()).trim().equals("==END==")){
	                sb.append(line);
	            }
	
	            System.out.println(Thread.currentThread().getName());
	            System.out.println("来自服务器的数据："+sb.toString());
	
	            br.close();
	            socket.close();
	        }catch (IOException e){
	            e.printStackTrace();
	        }
	
	    }
	}  
# 4、服务端  
	public static void runServer2() throws IOException {
        System.out.println("开始");
        ServerSocket ss=new ServerSocket(4444);//服务端监听3333这个端口
        while (true){
            Socket s=ss.accept();//服务端接受客户端的连接
            new Thread(new ServerThread(s)).start();
        }
        
    }  
  --  

	class ServerThread implements Runnable{
	    Socket s = null;
	    BufferedReader br = null;
	    public ServerThread(Socket s) throws IOException {
	        this.s = s;
	        br = new BufferedReader(new InputStreamReader(s.getInputStream()));
	    }
	
	    @Override
	    public void run() {
	        try {
	            InputStream in=s.getInputStream();//得到客户端的输入流，为了得到客户端传来的数据
	
	            OutputStream out =s.getOutputStream();//得到客户端的输出流，为了向客户端输出数据
	
	            BufferedReader bufferedReader=new BufferedReader(new InputStreamReader(in));
	
	            PrintWriter bufferedWriter=new PrintWriter(out,true);
	
	            String content = null;
	            StringBuffer sb = new StringBuffer("");
	            while (!(content = bufferedReader.readLine()).trim().equals("==END==")){
	                sb.append(content);
	            }
	            System.out.println("来自客户端的数据："+sb);
	            //bufferedWriter.println("我是来自服务端的回复，回复针对的是：");
	            bufferedWriter.println("我是来自服务端的回复，回复针对的是："+sb.toString().split(":")[1]+'\n'+"==END==");
	            bufferedReader.close();
	            s.close();
	        }catch (IOException e){
	            e.printStackTrace();
	        }
    	}
	}  
# 5、中转服务器   
	package socket.forward.StartServer;
	
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.boot.ApplicationArguments;
	import org.springframework.boot.ApplicationRunner;
	import org.springframework.stereotype.Component;
	import socket.forward.executor.impl.ForwarderImpl;
	
	import java.net.ServerSocket;
	import java.net.Socket;
	
	@Component
	public class ServerApplicationRunner implements ApplicationRunner {
	    @Autowired
	    ForwarderImpl forwarder;
	
	    @Override
	    public void run(ApplicationArguments args) throws Exception {
	        System.out.println("开始监听3574端口");
	        ServerSocket ss=new ServerSocket(3574);//服务端监听3574这个端口
	        while (true){
	            Socket s = ss.accept();
	            forwarder.execute(s);
	        }
	    }
	}  
 --  
	package socket.forward.executor.impl;
	
	
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.stereotype.Component;
	import socket.forward.SocketClient;
	import socket.forward.StartServer.Client;
	import socket.forward.executor.Forwarder;
	
	import java.io.*;
	import java.net.Socket;
	import java.util.concurrent.ExecutorService;
	
	@Component
	public class ForwarderImpl implements Forwarder {
	
	    private ExecutorService fixedThreadPool;
	
	    @Autowired
	    public void ForwarderImpl(ExecutorService fixedThreadPool){
	        this.fixedThreadPool = fixedThreadPool;
	    }
	    @Override
	    public void execute(Socket socket) {
	        fixedThreadPool.execute(new ForwarderTask(socket));
	    }
	
	    private class ForwarderTask implements Runnable {
	
	        private Socket socket;
	
	        ForwarderTask(Socket socket){
	            this.socket = socket;
	        }
	
	        @Override
	        public void run() {
	            try {
	                //向服务端建立连接
	                Socket s = new Socket("localhost", 4444);//创建一个客户端连接
	                System.out.println("成功创建一个客户端连接");
	
	                OutputStream outFS = s.getOutputStream();//向服务端写
	
	                InputStream inFS = s.getInputStream();//从服务端读
	                BufferedReader brFS = new BufferedReader(new InputStreamReader(inFS));
	                //BufferedWriter bwFS = new BufferedWriter(new OutputStreamWriter(outFS));
	                PrintStream psFS = new PrintStream(outFS);
	                //
	
	                PrintStream psFC = new PrintStream(socket.getOutputStream());	              
	
	                //从客户端读取的数据按行发送给服务端
	                String content = null;
	                BufferedReader br = new BufferedReader(new InputStreamReader(socket.getInputStream()));
	                while (!(content = br.readLine()).trim().equals("==END==") ){
	                    psFS.println(content);
	                }
	                psFS.println("==END==");
	                PrintStream psFC = new PrintStream(socket.getOutputStream());
	
	                //从服务端读取的数据按行发送给客户端
	                String con = null;
	                while ((con = brFS.readLine()).trim().equals("==END==")){
	                    psFC.println(con);
	                }
	                psFC.println("==END==");
	                //socket.shutdownOutput();
	                psFS.close();
	                psFC.close();
	                socket.close();
	                s.close();
	            }catch (IOException e){
	                e.printStackTrace();
	            }
	        }	   
	    }
	}



