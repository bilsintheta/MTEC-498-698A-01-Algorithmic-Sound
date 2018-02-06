# Session 02 - 01/30/18 - Basic Graphics / Audio Recording

<!-- MarkdownTOC -->

- [Session Overview](#session-overview)
- [Topics](#topics)
    - [Counter](#counter)
    - [Mouse Input](#mouse-input)
    - [Animate a Cube](#animate-a-cube)
    - [Basic Graphics](#basic-graphics)
    - [Audio Input](#audio-input)
    - [Basic Audio Recorder](#basic-audio-recorder)
- [Questions](#questions)
- [Homework](#homework)
- [Lab](#lab)

<!-- /MarkdownTOC -->


<a name="session-overview"></a>
## Session Overview

In the next two sessions, we'll explore the use of some basic graphics primitives, how to animate them, and use them to control the recording and playback of audio.  The end result will be multisampler that can record/playback audio, automatically segment audio, and we'll try and apply some simple filters.  Along the way we'll learn about a lot of basic coding principles including if statements, for loops, and how to use classes to build objects and abstract our code.  We'll also see how to use parameters to define elements in a graphical user interface.  By the end of today, we'll see how to create an audio recorder that can record audio and play it back, and next week we'll see how to add more interactivity including automatically segmenting audio and controlling the way it sounds with filtering.
 
<a name="topics"></a>
## Topics

<a name="counter"></a>
### Counter

```cpp
#include "ofMain.h"

class ofApp : public ofBaseApp{
public:
    void setup() {
        // do some initialization

        // set the size of the window
        ofSetWindowShape(250, 250);
        
        // the rate at which the program runs (FPS)
        ofSetFrameRate(30);
        
        // we are going to increment this variable each "frame"
        counter = 0;
        
        // create our font
        myFont.loadFont("dekar.ttf",    // name of the font file
                        36);            // size of the font
    }
    
    // i get called in a loop that runs until the program ends
    void update() {
        // update some variables
        counter = counter + 1;
    }
    
    // i also get called in a loop that runs until the program ends
    void draw() {
        // write the variable counter into a string
        char buf[80];
        sprintf(buf, "%d", counter);
        
        // draw the current counter
        myFont.drawString(buf, 100, 125);
    }

private:
    // declaration of variables
    int counter;
    ofTrueTypeFont myFont;
};


int main(){
    ofSetupOpenGL(1024, 768, OF_WINDOW);
    ofRunApp(new ofApp());
}
```


<a name="mouse-input"></a>
### Mouse Input

```cpp
#include "ofMain.h"

class ofApp : public ofBaseApp{
public:
    void setup() {
        // do some initialization
        
        // set the size of the window
        ofSetWindowShape(250, 250);
        
        // the rate at which the program runs (FPS)
        ofSetFrameRate(30);
        
        myFont.loadFont("dekar.ttf", 36);
        
        // don't clear the background on each update/draw loop
        // ofSetBackgroundAuto(false);
    }
    
    // i get called in a loop that runs until the program ends
    void update() {

    }
    
    // i also get called in a loop that runs until the program ends
    void draw() {
        // set the size of the circle
        float radius = 3.0f;
        
        // draw a circle at the mouse position
        ofCircle(mouseX, mouseY, radius);
        
        myFont.drawString("hello!", 50, 50);
        
        char buf[256];
        sprintf(buf, "%d %d", mouseX, mouseY);
        myFont.drawString(buf, 50, 125);
    }

    // void mousePressed(int x, int y, int button) { }

private:
    ofTrueTypeFont myFont;
    //bool bPressed;
};


int main(){
    ofSetupOpenGL(1024, 768, OF_WINDOW);
    ofRunApp(new ofApp());
}
```

<a name="animate-a-cube"></a>
### Animate a Cube

```cpp
#include "ofMain.h"

class ofApp : public ofBaseApp{
public:
    void setup() {
        // do some initialization
        
        // set the size of the window
        ofSetWindowShape(250, 250);
        
        // the rate at which the program runs (FPS)
        ofSetFrameRate(30);
        
        // we are going to increment this variable each "frame"
        counter = 0;
        
        brightness = 0;
        
        ofResetElapsedTimeCounter();
    }
    
    // i get called in a loop that runs until the program ends
    void update() {
        // update some variables
        counter = counter + 1;
    }
    
    // i also get called in a loop that runs until the program ends
    void draw() {
        ofBackground(0);
            
        brightness = sin(ofGetElapsedTimef() * 5.0) * 128.0 + 128.0;
        printf("%f\n", brightness);
        ofSetColor(brightness, 0, 0);
        ofNoFill();
        ofRect(20, 20, 100, 100);
    }


private:
    int counter;
    float brightness;
};


int main(){
    ofSetupOpenGL(1024, 768, OF_WINDOW);
    ofRunApp(new ofApp());
}
```

<a name="basic-graphics"></a>
### Basic Graphics

```cpp
#include "ofMain.h"

class ofApp : public ofBaseApp{
public:
    void setup() {
        // do some initialization
        
        // set the size of the window
        ofSetWindowShape(width, height);
        
        // the rate at which the program runs (FPS)
        ofSetFrameRate(30);
        
        // don't clear the background on each update/draw loop
//         ofSetBackgroundAuto(false);
    }
    
    // i get called in a loop that runs until the program ends
    void update() {

    }
    
    // i also get called in a loop that runs until the program ends
    void draw() {
        // clear the background to black
        ofBackground(0);
        
        float frequency = 2.0;
        float amplitude = height / 2.0;
        float phase = ofGetElapsedTimef();
        
        ofSetColor(100, 100, 100);
        ofDrawLine(0, height / 2, width, height / 2);
        
        ofSetColor(200, 200, 200);
        for (int i = 1; i < width; i++) {
            
            float x1 = (float)i / width;
            float y1 = amplitude * sin( phase + frequency * x1 * 2.0*PI );
            float x2 = (float)(i - 1) / width;
            float y2 = amplitude * sin( phase + frequency * x2 * 2.0*PI );
            
            ofDrawLine(x1 * width, -y1 + height / 2.0,
                       x2 * width, -y2 + height / 2.0);
        }
    }
    
    void keyPressed(int key) {
        if (key == 'f') {
            ofToggleFullscreen();
        }
    }
    
    void mouseMoved( int x, int y ){

    }

private:
    int width = 640;
    int height = 480;
};


int main(){
    ofSetupOpenGL(1024, 768, OF_WINDOW);
    ofRunApp(new ofApp());
}
```

<a name="audio-input"></a>
### Audio Input

```cpp
#include "ofMain.h"

class ofApp : public ofBaseApp{
public:
    void setup() {
        // do some initialization
        
        // the rate at which the program runs (FPS)
        ofSetFrameRate(30);
        
        width = 480;
        height = 320;
        ofSetWindowShape(width, height);
        
        sampleRate = 44100;
        initialBufferSize = 512;
        buffer.resize(initialBufferSize);
        ofSoundStreamSetup(0, 1, this, sampleRate, initialBufferSize, 4);
    }
    
    // i get called in a loop that runs until the program ends
    void update() {
        
    }
    
    // i also get called in a loop that runs until the program ends
    void draw() {
        
        ofBackground(0);
        
        // change future drawing commands to be white
        ofSetColor(255, 255, 255, 200);

        // move the drawing down halfway
        ofTranslate(0, height/2);
        
        // this is going to store how many pixels we multiply our drawing by
        float amplitude = 100;
        
        // conversion from samples to pixels
        float width_ratio = width / (float)(initialBufferSize);
        
        // draw our audio buffer
        for (int i = 1; i < initialBufferSize; i++)
        {
            ofLine((i - 1) * width_ratio,
                   buffer[(i-1)] * amplitude,
                   i * width_ratio,
                   buffer[i] * amplitude);
        }
        
    }
    
    void keyPressed(int key) {
        if (key == 'f') {
            ofToggleFullscreen();
        }
    }
    

    void audioIn(float * input, int bufferSize, int nChannels) {
        
        if( initialBufferSize != bufferSize ){
            ofLog(OF_LOG_ERROR, "your buffer size was set to %i - but the stream needs a buffer size of %i", initialBufferSize, bufferSize);
            return;
        }

        // let's add the current frame of audio input to our recording buffer.  this is 512 samples.
        // (note: another way to do this is to copy the whole chunk of memory using memcpy)
        for (int i = 0; i < bufferSize; i++)
        {
            // we will set the value at each sample
            buffer[i] = input[i];
        }

    }
    

private:
    int                 width,
                        height;
    
    // variables for audio
    int                 initialBufferSize,
                        sampleRate;
    
    // this vector will store our audio recording
    vector<float>       buffer;

};


int main(){
    ofSetupOpenGL(1024, 768, OF_WINDOW);
    ofRunApp(new ofApp());
}

```

<a name="basic-audio-recorder"></a>
### Basic Audio Recorder

```cpp
#include "ofMain.h"


    // declare the padbutton class and methods
class padButton {
public:
    // enumerators allow us to assign more interesting names to values of an integer
    // we could use an integer to the same effect,
    // e.g. "int button_state = 0", when our button is down
    // and "int button_state = 1", when our button is normal,
    // but enumerators allow us to instead say
    // "BUTTON_STATE button_state = BUTTON_DOWN", when our button is down,
    // "BUTTON_STATE button_state = NORMAL", when our button is normal.
    enum BUTTON_STATE {
        BUTTON_DOWN,
        NORMAL
    };
    
    // default constructor definition
    padButton()
    {
        // by default, we have a very boring button
        button_x = 0;
        button_y = 0;
        button_width = 0;
        button_height = 0;
        button_state = NORMAL;
    }

    // methods which our button class will define
    // one for loading images for each of the button states
    void load(string state_normal, string state_down)
    {
        // load the images for our buttons
        button_image_normal.load(state_normal);
        button_image_down.load(state_down);
    }
    
    // setters, to set internal variables
    // the position
    void setPosition(int x, int y)
    {
        // set our internal variables
        button_x = x;
        button_y = y;
    }
    
    // the size
    void setSize(int w, int h)
    {
        // set our internal variables
        button_width = w;
        button_height = h;
    }

    // drawing the buttons
    void draw()
    {
        // allow alpha transparency
        ofEnableAlphaBlending();
        
        // if our button is normal
        if(button_state == NORMAL)
        {
            // draw the normal button image
            button_image_normal.draw(button_x,
                                     button_y,
                                     button_width,
                                     button_height);
        }
        else
        {
            // draw the down image
            button_image_down.draw(button_x,
                                   button_y,
                                   button_width,
                                   button_height);
        }
        
        // ok done w/ alpha blending
        ofDisableAlphaBlending();
    }

    // and interaction with the button
    bool pressed(int x, int y)
    {
        // compound boolean expressions to determine,
        // is our x,y input within the bounds of the button
        // we have to check the left, the top, the right, and the bottom sides
        // of the button, respectively.
        if( x > button_x && y > button_y
           && x < (button_x + button_width)
           && y < (button_y + button_height) )
        {
            button_state = BUTTON_DOWN;
            
            // return yes since the user pressed the button
            return true;
        }
        else {
            // no the user didn't press the button
            return false;
        }
    }

    bool released(int x, int y)
    {
        // ok back to normal
        button_state = NORMAL;
        
        // we always return true since that is how our buttons work.
        return true;
    }
private:
    
    // images for drawing
    ofImage button_image_normal, button_image_down;
    
    // our position
    int button_x, button_y;
    
    // size
    int button_width, button_height;
    
    // and internal state of the button
    BUTTON_STATE button_state;
};

class ofApp : public ofBaseApp{
public:
    void setup() {
        // do some initialization
        
        // the rate at which the program runs (FPS)
        ofSetFrameRate(30);
        
        // we initialize our two buttons to act as a play and a record button.
        button_record.load("button-record.png", "button-record-down.png");
        button_play.load("button-play.png", "button-play-down.png");
        button_record.setPosition(100, 110);
        button_record.setSize(100, 100);
        button_play.setPosition(250, 110);
        button_play.setSize(100, 100);
        
        // initially not recording
        bRecording = false;
        bPlaying = false;
        bRecorded = false;
        
        // which audio frame am i currently playing back
        frame = 0;
        
        // how many audio frames did we record?
        numFrames = 0;
        
        width = 480;
        height = 320;
        ofSetWindowShape(width, height);
        
        sampleRate = 44100;
        initialBufferSize = 512;
        ofSoundStreamSetup(1, 1, this, sampleRate, initialBufferSize, 4);
    }
    
    // i get called in a loop that runs until the program ends
    void update() {
        
    }
    
    // i also get called in a loop that runs until the program ends
    void draw() {
        
        ofBackground(0);
        
        // draw our buttons

        // change future drawing commands to be white
        ofSetColor(255, 255, 255, 200);
        
        button_record.draw();
        button_play.draw();
        
        if(bRecorded)
        {
                // move the drawing down halfway
            ofTranslate(0, height/2);
            
                // this is going to store how many pixels we multiply our drawing by
            float amplitude = 100;
            
                // conversion from samples to pixels
            float width_ratio = width / (float)(initialBufferSize*numFrames);
            
                // draw our audio buffer
            for (int i = 1; i < initialBufferSize*numFrames; i++)
            {
                ofLine((i - 1) * width_ratio,
                       buffer[(i-1)] * amplitude,
                       i * width_ratio,
                       buffer[i] * amplitude);
            }
        }
        
    }
    
    void keyPressed(int key) {
        if (key == 'f') {
            ofToggleFullscreen();
        }
    }
    
    void mouseMoved( int x, int y ){
        
    }
    
    void mousePressed(int x, int y, int button) {
        bPlaying = button_play.pressed(x, y);
        bRecording = button_record.pressed(x, y);
    }
    
    void mouseReleased(int x, int y, int button) {
        // we then user stops pressing the button, we are no longer playing or recording
        if(button_play.released(x, y))
            bPlaying = false;
        if(button_record.released(x, y))
            bRecording = false;
    }
    
    void audioOut(float * output, int bufferSize, int nChannels) {
        
            // if we are playing back audio (if the user is pressing the play button)
        if(bPlaying && bRecorded)
        {
                // we set the output to be our recorded buffer
            for (int i = 0; i < bufferSize; i++){
                    // so we have to access the current "playback frame" which is a variable
                    // "frame".  this variable helps us determine which frame we should play back.
                    // because one frame is only 512 samples, or 1/90th of a second of audio, we would like
                    // to hear more than just that one frame.  so we playback not just the first frame,
                    // but every frame after that... after 90 frames of audio, we will have heard
                    // 1 second of the recording...
                output[i] = buffer[i + frame*bufferSize];
            }
            
                // we have to increase our frame counter in order to hear farther into the audio recording
            frame = (frame + 1) % numFrames;
            
        }
            // else don't output anything to the speaker
        else {
            memset(output, 0, nChannels * bufferSize * sizeof(float));
        }
    }

    void audioIn(float * input, int bufferSize, int nChannels) {
        
        if( initialBufferSize != bufferSize ){
            ofLog(OF_LOG_ERROR, "your buffer size was set to %i - but the stream needs a buffer size of %i", initialBufferSize, bufferSize);
            return;
        }
        
            // if we are recording
        if(bRecording)
        {
                // let's add the current frame of audio input to our recording buffer.  this is 512 samples.
                // (note: another way to do this is to copy the whole chunk of memory using memcpy)
            for (int i = 0; i < bufferSize; i++)
            {
                    // we will add a sample at a time to the back of the buffer, increasing the size of "buffer"
                buffer.push_back(input[i]);
            }
            
                // we also need to keep track of how many audio "frames" we have.  this is how many times
                // we have recorded a chunk of 512 samples.  we refer to that chunk of 512 samples as 1 frame.
            numFrames++;
            
            bRecorded = true;
        }
            // otherwise we set the input to 0
        else
        {
                // set the chunk in memory pointed to by "input" to 0.  the
                // size of the chunk is the 3rd argument.
            memset(input, 0, nChannels * bufferSize * sizeof(float));
        }
        
    }
    

private:
    int                 width,
                        height;
    
    // variables for audio
    int                 initialBufferSize,
                        sampleRate;
    
    // this vector will store our audio recording
    vector<float>       buffer;
    
    // frame will tell us "what chunk of audio are we currently playing back".
    // as we record and play back in "chunks" also called "frames", we will need
    // to keep track of which frame we are playing during playback
    int                 frame;
    
    // frame will run until it hits the final recorded frame, which is numFrames
    // we increment this value every time we record a new frame of audio
    int                 numFrames;
    
    // our buttons for user interaction
    padButton           button_play, button_record;
    
    // determined based on whether the user has pressed the play or record buttons
    bool                bRecording, bPlaying, bRecorded;
    
    // single instance of our padButton class
    // padButton           button1;
    // a vector is a c-standard library implementation of an array
    // this allows us to create multiple buttons
    vector<padButton>   buttons;
};


int main(){
    ofSetupOpenGL(1024, 768, OF_WINDOW);
    ofRunApp(new ofApp());
}

```

<a name="questions"></a>
## Questions

<a name="homework"></a>
## Homework

Extend the "Basic Audio Recorder" example.

As an example, consider making a multisampler like an MPC style array of buttons where each pad can be recorded and played back individually.  If you choose to do this, you may want to explore the use of the vector object (e.g., vector<padButton> buttons), or use an array (e.g. padButton buttons[10]), or try just simply creating many buttons with the name, "button1", "button2", etc...  Each button should be responsible for its own interaction and recording/storage/playback of audio, though all behavior should be moderated/mediated by the main class, ofApp.

<a name="lab"></a>
## Lab

Start on the homework.
