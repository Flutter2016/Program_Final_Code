# Program_Final_Code

// A Less Simple Carnivore Client
//
// Note: requires Carnivore Library for Processing (http://r-s-g.org/carnivore)
//
// + Windows people:  first install winpcap (http://winpcap.org)
// + Mac people:      first open a Terminal and execute this commmand: sudo chmod 777 /dev/bpf*
//                    (must be done each time you reboot your mac)


import java.util.Iterator;
import org.rsg.carnivore.*;
import org.rsg.carnivore.net.*;
import org.rsg.lib.Log;

HashMap nodes = new HashMap();
float startDiameter = 150.0;
float shrinkSpeed = 0.99;
int splitter, x, y;
CarnivoreP5 c;
PFont font32;
PImage terminalInFinder;
PImage terminalSudoCommand;


import ddf.minim.* ;
import ddf.minim.signals.* ;
import ddf.minim.effects.* ;
Minim minim;

AudioOutput au_out ;
SquareWave sqw ;
LowPassSP lpass ;


void setup(){
  size(800, 600);
  ellipseMode(CENTER);
  
 
  
  terminalInFinder = loadImage("terminalInFinder.png");
  terminalSudoCommand = loadImage("terminalSudoCommand.png");

  Log.setDebug(true); // Uncomment this for verbose mode
  c = new CarnivoreP5(this);
  //c.setVolumeLimit(4);
  
  
  
  minim = new Minim(this) ;
 au_out = minim.getLineOut() ;

 // create a SquareWave with a frequency of 440 Hz,
 // an amplitude of 1 and the same sample rate as out
 sqw = new SquareWave(440, 1, 44100);
 // create a LowPassSP filter with a cutoff frequency of 200 Hz
 // that expects audio with the same sample rate as out
 lpass = new LowPassSP(200, 44100);
 // now we can attach the square wave and the filter to our output
 au_out.addSignal(sqw);
 au_out.addEffect(lpass);
  
  
}

void draw(){
  if(c.isMacAndPromiscuousModeFailed) {
    drawError();
  } else {
    drawMap();
  }
}

void drawMap(){
  background(255);
  drawNodes();
 
}

void drawError(){
  int x = width/2 - 200; 
  int y = 75;
  int lineheight = 32; 
  
  background(255);
  fill(0, 102, 153);
  text("Please initialize packet sniffing.", x, y);
  y += lineheight*2;
  
  text("Step 1--Open the Terminal.", x, y);
  y += 20;
  
  image(terminalInFinder, x, y);
  y += lineheight*4.5;

  text("Step 2--Type this command:", x, y);
  y += lineheight;

  fill(0);
  text("sudo chmod 777 /dev/bpf*", x, y);
  y += 10;

  image(terminalSudoCommand, x-33, y);
  y += lineheight*6;

  fill(0, 102, 153);
  text("Step 3--Quit and relaunch.", x, y);
  y += lineheight;
  
}

// Iterate through each node 
synchronized void drawNodes() {
  Iterator it = nodes.keySet().iterator();
  while(it.hasNext()){
    String ip = (String)it.next();
    float d = float(nodes.get(ip).toString());

    // Use last two IP address bytes for x/y coords
    String ip_as_array[] = split(ip, '.');
    x = int(ip_as_array[2]) * width / 255; // Scale to applet size
    y = int(ip_as_array[3]) * height / 255; // Scale to applet size
    
    // Draw the node
    stroke(0);
     fill(random(255),random(255),random(255)); // Rim
    ellipse(x, y, d, d);             // Node circle
    noStroke();
     fill(random(255),random(255),random(255));  // Halo
    ellipse(x, y, d + 20, d + 20);
    
    // Shrink the nodes a little
    if(d > 50)
      nodes.put(ip, str(d * shrinkSpeed));
  }  
   
}

// Called each time a new packet arrives
synchronized void packetEvent(CarnivorePacket packet){
  println("[PDE] packetEvent: " + packet);

  // Remember these nodes in our hash map
  nodes.put(packet.receiverAddress.toString(), str(startDiameter));
  nodes.put(packet.senderAddress.toString(), str(startDiameter));
  
  float r = random(1000);
  sqw.setFreq(packet.receiverPort) ;
 frameRate(10);
}

void keyPressed()
{
 if ( key == 'm' )
 {
 if ( au_out.isMuted() )
 {
 au_out.unmute();
 }

 else
 {
 au_out.mute();
 }
 }
}
