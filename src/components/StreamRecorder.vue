<template>
    <div>
        <h2>WebRTC functionality</h2>
        <hr>
        <video class="media_v" id="webcamVideo" autoplay playsinline></video>
    <div > 
        <button @click="startWebcam()">
        Start Streaming 
        </button> 
        
        <button @click="stopStream()">
        Stop Streaming
        </button> 
        
        <button @click="screenShot()">
        Screen Shot
        </button> 
        
        <button @click="recordStream()" 
        id="startRecord"
        >
        Start recording 
        </button>
        
        <button 
        @click="stopRecording()" 
        id="playRecord"
        >
        Stop Recording 
        </button>
        
        <button 
        @click="playStream()" 
        id="playRecord"
        >
        Play 
        </button>      
    </div>
    <p style="color: red" v-if="this.recording">it's recording ...</p>
    <div v-if="this.iscaptured">
        <h3> Captured Screen</h3>
    </div>
    <canvas id="canvas"> </canvas>
    <div v-if="this.isrecordedVideo">
        <h2>Recorded Video</h2>
    </div>
    <video class="media_v" id="streamVideo" autoplay playsinline></video>            
</div>
</template>

<script>
export default {
    name: 'WebRTC',
    data :function() {
    return {
        db: null,
        servers: {
        iceServers: [
            {
                urls: [
                    "stun:stun1.l.google.com:19302",
                ],
            },
        ],
            iceCandidatePoolSize: 10,
        },
        mediaStream: null,
        rtcPeerConnection: null,
        stream: null,
        webcamVideo: null,
        streamVideo: null,
        mediaRecorder: null,
        recordedVideo: [],
        startRecord: null,
        playRecord: null,
        iscaptured: false,
        isrecordedVideo: false,
        recording: false,
        canvas: null
        }   
    },
    async created() {

    },
    mounted() {
        this.rtcPeerConnection = new RTCPeerConnection(this.servers);
    // HTML elements
        this.webcamVideo = document.getElementById("webcamVideo");
        this.streamVideo = document.getElementById("streamVideo");
        this.startRecord = document.getElementById('startRecord');
        this.playRecord = document.getElementById('playRecord');
        // elements to get screenshot
        this.shotScreen = document.getElementById('screenShot')
        this.canvas = document.getElementById('canvas')

    },
    methods: {
        startWebcam: function(){
            this.stream = navigator.mediaDevices.getUserMedia({
                video: {
                width: { max: 1020},
                height: { max: 320 }
            },
                audio: true
            }).then(function(mediaStream){
                webcamVideo.srcObject = mediaStream
                webcamVideo.onloadedmetadata = function(e){
                webcamVideo.play()
                }
            })
            .catch(function(error){
                alert(error)
            })
        },
        stopStream: function(){
            if(this.webcamVideo.srcObject){
                this.webcamVideo.srcObject.getTracks().forEach((track)=>{
                    track.stop()
                })
            }
        },        
        recordStream: function(){
            this.recordedVideo = []
            let videoRec = []
            videoRec = this.recordedVideo 
            var options = {
                audioBitsPerSecond : 128000,
                videoBitsPerSecond : 2500000,
                mimeType : 'video/webm;codecs=vp8,opus'
            }

            let videoStream = this.webcamVideo.srcObject    
            if(videoStream !=null){
                this.mediaRecorder = new MediaRecorder(videoStream , options)
                console.log('the mediarecorder'+this.mediaRecorder)
                this.recording = true
                this.isrecordedVideo = true
                this.mediaRecorder.ondataavailable = function(event){
                    if(event.data && event.data.size > 0) {
                        videoRec.push(event.data)
                        return
                    }
                }
                this.mediaRecorder.start()
            }else{
                alert('there is no streaming pls start streaming first')
            }
        },
        playStream: function(){
            if(this.mediaRecorder != null){
                const superBuffer = new Blob(this.recordedVideo);
                this.streamVideo.src = URL.createObjectURL(superBuffer);
                this.streamVideo.controls = true;
                this.streamVideo.play();
            }else{
                alert ('there is no recorded video to be displayed')
            }
        },
        stopRecording: function(){
            if(this.mediaRecorder != null){
                this.mediaRecorder.stop();
                this.recording = false 
            }else{
                alert('it is not recording')
            }
        },
        screenShot: function(){
            if(this.webcamVideo){
                this.iscaptured = true
                this.canvas.width = this.webcamVideo.videoWidth
                this.canvas.height = this.webcamVideo.videoHeight
                this.canvas.getContext('2d').drawImage(this.webcamVideo, 0, 0 , 320, 320)
            }else {
                alert('error')
            }  
        },
    }
}
</script>

<style>
.disabled{
    display: none;
    background-color: darkgray;
}
</style>