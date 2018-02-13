# Session 03 - 02/06/18 - Audio Interaction

<!-- MarkdownTOC -->

- [Session Overview](#session-overview)
- [Topics](#topics)
    - [RMS](#rms)
    - [Audio Segmentation](#audio-segmentation)
    - [Lab](#lab)
    - [Homework](#homework)

<!-- /MarkdownTOC -->


<a name="session-overview"></a>
## Session Overview

We'll continue with record/playback audio, automatically segment audio, and apply some random playback to our detected grains.
 
<a name="topics"></a>
## Topics

<a name="rms"></a>
### RMS

```cpp
#include "ofMain.h"


class App : public ofBaseApp{
public:
    void setup() {
        width = 640;
        height = 480;
        ofSetWindowShape(width, height);
        
        ofSetFrameRate(60);
        ofSetBackgroundAuto(false);
        
        sample_rate = 44100;
        buffer_size = 512;
        audio_input.resize(buffer_size);
        ofSoundStreamSetup(0, 1, sample_rate, buffer_size, 3);
    }
    
    // i get called in a loop that runs until the program ends
    void update() {
        
    }
    
    // i also get called in a loop that runs until the program ends
    void draw() {
        ofEnableAlphaBlending();
        ofSetColor(0, 0, 0, 10);
        ofDrawRectangle(0, 0, width, height);
        
        // draw a line across the middle of the screen
        ofSetColor(100, 100, 100);
        ofDrawLine(0, height / 2, width, height / 2);
        
        // we draw the audio input as before
        ofSetColor(200, 200, 200);
        float amplitude = 100.0f;
        for (int i = 1; i < audio_input.size(); i++) {
            
            // get two pairs of points
            float x1 = i * width / (float)audio_input.size();
            float y1 = amplitude * audio_input[i];
            float x2 = (i - 1) * width / (float)audio_input.size();
            float y2 = amplitude * audio_input[i-1];
            
            // draw a tiny segment of the overall line
            ofDrawLine(x1, -y1 + height / 2,
                       x2, -y2 + height / 2);
        }
        
        // draw a circle in the middle of the screen with the size
        // set by the rms value
        ofSetRectMode(OF_RECTMODE_CENTER);
        ofDrawCircle(width / 2, height / 2, rms * width);

        if(rms_values.size() > 0)
        {
            ofSetRectMode(OF_RECTMODE_CORNER);
            
            for (int i = 1; i < rms_values.size(); i++) {
                ofDrawLine(i * width / (float)rms_values.size(),
                           -rms_values[i] * 1000.0 + height,
                           (i - 1) * width / (float)rms_values.size(),
                           -rms_values[i - 1] * 1000.0 + height);
            }

            // calculate the average of the rms values
            float average_rms = 0.0f;
            for (int i = 0; i < rms_values.size(); i++) {
                average_rms = average_rms + rms_values[i];
            }
            average_rms = average_rms / rms_values.size();
            
            // calculate the variance of the rms values
            float var_rms = 0.0f;
            for (int i = 0; i < rms_values.size(); i++) {
                var_rms = var_rms + abs(rms_values[i] - average_rms);
            }
            var_rms = var_rms / rms_values.size();
            
            // now we see if the current value is outside the mean + variance
            // basic statistics tells us a normally distributed function
            // has a mean and a variance where 97% of the data is explained by
            // 3 standard deviations.  we use this principle here in detecting 
            // the the current rms reading is outside this probability
            float min_rms = 0.01;
            if (rms_values.back() > (average_rms + 2.0 * var_rms) &&
                rms_values.back() > min_rms) {
                // draw a rectangle to denote the detection of an onset
                ofDrawRectangle(0, 0, width, height);
            }
        }
        ofDisableAlphaBlending();
    }
    
    void audioIn(float * input, int buffer_size, int n_channels)
    {
        // copy the data into our variable, audioInput
        std::memcpy(&audio_input[0], input, sizeof(float) * buffer_size);
        
        // add all the audio input values
        float total = 0;
        for (int i = 0; i < buffer_size; i++) {
            // we add the "square" of each value so that negative numbers
            // become positive.  this is like thinking of the "magnitude"
            total = total + (input[i] * input[i]);
        }
        // the "mean" part of the RMS, we divide by the number of audio input samples
        // we added in the for loop above
        total = total / (float)buffer_size;
        
        // the "root" part of RMS, we take the square root to get our RMS reading for the
        // current chunk of audio input values
        rms = sqrt(total);
        
        // add the current rms value
        rms_values.push_back(rms);
        
        // we only keep a maximum of 1 second of the rms readings
        if (rms_values.size() > (sample_rate / buffer_size)) {
            // then we delete the first one
            rms_values.erase(rms_values.begin(), rms_values.begin() + 1);
        }
    }
    

private:
    int                     width,
                            height;
    
    vector<float>           audio_input;
    
    int                     sample_rate,
                            buffer_size;
    
    float                   rms;
    
    vector<float>           rms_values;
};


int main(){
    ofSetupOpenGL(1024, 768, OF_WINDOW);
    ofRunApp(new App());
}
```

<a name="audio-segmentation"></a>
### Audio Segmentation

```cpp
#include "ofMain.h"


class App : public ofBaseApp{
public:
    void setup() {
        width = 640;
        height = 480;
        ofSetWindowShape(width, height);
        
        ofSetFrameRate(60);
        ofSetBackgroundAuto(false);
        
        sample_rate = 44100;
        buffer_size = 256;
        
        min_samples_per_grain = 0.2 * sample_rate;
        current_frame_i = 0;
        current_grain_i = 0;
        total_samples = 0;
        
        audio_input.resize(buffer_size);
        rms_values.resize(sample_rate / buffer_size);
        ofSoundStreamSetup(1, 1, sample_rate, buffer_size, 3);
    }
    
    // i get called in a loop that runs until the program ends
    void update() {
        
    }
    
    // i also get called in a loop that runs until the program ends
    void draw() {
        ofBackground(0);
        
        if(b_recording) {
            ofDrawBitmapString("Recording", 20, 20);
        }
        else {
            ofDrawBitmapString("Playing", 20, 20);
        }
        ofDrawBitmapString("Grains: " + ofToString(grains.size()), 20, 40);
        ofDrawBitmapString("Current Grain: " + ofToString(current_grain_i), 20, 60);
        ofDrawBitmapString("Current Frame: " + ofToString(current_frame_i), 20, 80);
        
        float skip = 4;
        float width_step = width / (float)(total_samples / skip);
        int samp_i = 0;
        for(int i = 0; i < grains.size(); i++) {
            bool this_grain = false;
            int prev_grain_start = samp_i;
            for(int j = 1; j < grains[i].size(); j+=4) {
                if (i == current_grain_i && j >= current_frame_i * buffer_size && j <= (current_frame_i + 1) * buffer_size) {
                    this_grain = true;
                    ofSetColor(255, 100, 100, 30);
                    ofDrawLine(samp_i * width_step, 0, (samp_i + 1) * width_step, height);
                }
                ofSetColor(255);
                ofLine(samp_i * width_step, -grains[i][j - 1] * height / 2 + height / 2,
                       (samp_i + 1) * width_step, -grains[i][j] * height / 2 + height / 2);
                samp_i++;
            }
            if(this_grain) {
                ofSetColor(100, 255, 100, 30);
                ofSetRectMode(OF_RECTMODE_CORNER);
                ofDrawRectangle(prev_grain_start * width_step, 0,
                                (samp_i - prev_grain_start) * width_step, height);
            }
        }
    }
    
    void keyPressed(int k) {
        if(k == ' '){
            b_recording = !b_recording;
        }
    }
    
    void audioOut(float * output, int buffer_size, int n_channels)
    {
        if (b_recording) {
            return;
        }
        
        if(grains.size()) {
            
            for (int i = 0; i < buffer_size; i++) {
                output[i] = grains[current_grain_i][i + current_frame_i * buffer_size];
            }
            
            current_frame_i += 1;
            
            if(current_frame_i * buffer_size > min_samples_per_grain ||
               current_frame_i >= grains[current_grain_i].size() / buffer_size) {
                if(rand() % 100 > 75) {
                    current_frame_i = 0;
                }
                else {
                    current_grain_i = rand() % grains.size();
                    current_frame_i = 0;
                }
            }
        }
    }
    
    void audioIn(float * input, int buffer_size, int n_channels)
    {
        // copy the data into our variable, audioInput
        std::memcpy(&audio_input[0], input, sizeof(float) * buffer_size);

        if (!b_recording) {
            return;
        }
        
        // add all the audio input values
        float total = 0;
        for (int i = 0; i < buffer_size; i++) {
            // we add the "square" of each value so that negative numbers
            // become positive.  this is like thinking of the "magnitude"
            total = total + (input[i] * input[i]);
        }
        // the "mean" part of the RMS, we divide by the number of audio input samples
        // we added in the for loop above
        total = total / (float)buffer_size;
        
        // the "root" part of RMS, we take the square root to get our RMS reading for the
        // current chunk of audio input values
        rms = sqrt(total);
        
        // calculate the average of the rms values
        float average_rms = 0.0f;
        for (int i = 0; i < rms_values.size(); i++) {
            average_rms = average_rms + rms_values[i];
        }
        average_rms = average_rms / rms_values.size();
        
        // calculate the variance of the rms values
        float var_rms = 0.0f;
        for (int i = 0; i < rms_values.size(); i++) {
            var_rms = var_rms + abs(rms_values[i] - average_rms);
        }
        var_rms = var_rms / rms_values.size();
        
        // now we see if the current value is outside the mean + variance
        // basic statistics tells us a normally distributed function
        // has a mean and a variance where 97% of the data is explained by
        // 3 standard deviations.  we use this principle here in detecting
        // the the current rms reading is outside this probability
        float min_rms = 0.01;
        if (rms > (average_rms + 2.0 * var_rms) &&
            rms > min_rms &&
            current_grain.size() > min_samples_per_grain) {
            grains.push_back(current_grain);
            current_grain.resize(0);
        }
        
        for (int i = 0; i < buffer_size; i++) {
            current_grain.push_back(input[i]);
        }
        total_samples += buffer_size;
        
        // add the current rms value
        rms_values.push_back(rms);
        
        // we only keep a maximum of 1 second of the rms readings
        if (rms_values.size() > (sample_rate / buffer_size)) {
            // then we delete the first one
            rms_values.erase(rms_values.begin(), rms_values.begin() + 1);
        }

    }
    

private:
    int                     width,
                            height;
    
    vector<float>           audio_input;
    
    int                     sample_rate,
                            buffer_size;
    
    float                   rms;
    
    vector<float>           rms_values;
    
    vector<vector<float>>   grains;
    vector<float>           current_grain;
    int                     total_samples;

    int                     current_frame_i;
    int                     current_grain_i;
    int                     min_samples_per_grain;
    
    bool                    b_recording;
};


int main(){
    ofSetupOpenGL(1024, 768, OF_WINDOW);
    ofRunApp(new App());
}
```

<a name="lab"></a>
### Lab

Work through the code and make sure you understand how RMS is used to create grains, how they are stored in `vectors`, and how you could potentially interact with the process of playing back those grains.

<a name="homework"></a>
### Homework

Work on your project.  You will have two weeks to develop your project and will present it in class in 2 weeks.  You will have 4 minutes, no more.  Spend two minutes to demonstrate your work, and 2 minutes to explain.

You should think about how you can build in audio interaction and procedurally generate sound/music using the few techniques we've discussed: RMS, Onset Detection, Grains, and Random playback.  What ideas/processes come to mind with these techniques?  Integrate them into your existing musical/sound practice, and explore its limits/advantages.  If you are feeling stuck, look at the other openFrameworks examples and see what other influences/processes you may want to explore.  Could you imagine for instance a webcam interacting with the process?  A physical game environment?  Perhaps a human performer that plays with a simple generative instrument/vocal sampler?  Perhaps you also want to explore the parameters of this process: threshold, grain size, frequency of playback, silence/timing, additional real-time signals that you could analyze for onsets (e.g. video... RMS works the same way for pixels as it does audio).
