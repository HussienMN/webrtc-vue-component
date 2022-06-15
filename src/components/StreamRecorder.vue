<template>
    <div>

            <h2>Streaming Video</h2>

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
                        :style="{textContent}"
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
                <button @click="downloadStream()" 
                        id="downloadRecord">
                Download
                </button>                                                      
            
            </div>
            <h3> Captured Screen</h3>
            <canvas id="canvas"> </canvas>
            <h2>Recorded Video</h2>
            <video class="media_v" id="remoteVideo" autoplay playsinline></video>
            
</div>
</template>

<script>
// import myDB from '../firebase'
// import firebase from 'firebase/compat/app';
// import 'firebase/compat/firestore';
// import 'firebase/compat/analytics';
// import { collection, query, where, onSnapshot } from "firebase/firestore";

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
        localStream: null,
        mediaStream: null,
        pc: null,
        stream: null,
        webcamVideo: null,
        remoteVideo: null,
        mediaRecorder: null,
        recordedVideo: [],
        startRecord: null,
        playRecord: null,
        downloadRecord:null,
        textContent: 'Start Recording',
        canvas: null
        }   
    },
    async created() {

    },
    mounted() {
        this.pc = new RTCPeerConnection(this.servers);
    // HTML elements
        this.webcamVideo = document.getElementById("webcamVideo");
        this.remoteVideo = document.getElementById("remoteVideo");
        this.startRecord = document.getElementById('startRecord');
        this.playRecord = document.getElementById('playRecord');
        this.downloadRecord = document.getElementById('downloadRecord');
        // elements to get screenshot
        this.shotScreen = document.getElementById('screenShot')
        this.canvas = document.getElementById('canvas')
        this.canvas.width = 320
        this.canvas.height = 320
    },
    methods: {
        stopStream: function(){

            if(this.webcamVideo.srcObject){
                this.webcamVideo.srcObject.getTracks().forEach((track)=>{
                    track.stop()
                })
            }
        },
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
            }).catch(function(error){
                alert(error)
            })
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
            this.remoteVideo.src = URL.createObjectURL(superBuffer);
            this.remoteVideo.controls = true;
            this.remoteVideo.play();
        }else{
            alert ('there is no recorded video to be displayed')
        }
    },
    stopRecording: function(){
        if(this.mediaRecorder != null){
            this.mediaRecorder.stop();
            alert('stopped !')
        }else{
            alert('it is not recording')
        }
    },
    downloadStream: function(){
        
        if(this.mediaRecorder !=null){
            const blob = new Blob(this.recordedVideo, {type: 'video/webm;codecs=vp8,opus'});
            const url = window.URL.createObjectURL(blob);
            setTimeout(() => {
                window.URL.revokeObjectURL(url);
            }, 100)
        }else {
            alert('there is nothing to download')
        }
    },
    screenShot: function(){
        if(this.webcamVideo){
            this.canvas.width = this.webcamVideo.videoWidth
            this.canvas.height = this.webcamVideo.videoHeight
            this.canvas.getContext('2d').drawImage(this.webcamVideo, 0, 0 , this.canvas.width, this.canvas.height)
        }else {
            alert('error')
        }  
    },
    async createCall() {
    //   // Firestore calls
    //     const callDoc = this.db.collection("calls").doc();
    //     const offerCandidates = callDoc.collection("offerCandidates");
    //     const answerCandidates = callDoc.collection("answerCandidates");

    //     // Candidate
    //     this.pc.onicecandidate = (event) => {
    //         event.candidate && offerCandidates.add(event.candidate.toJSON());
    //     };

    //     // offer
    //     const offerDescription = await this.pc.createOffer();
    //     await this.pc.setLocalDescription(offerDescription);

    //     const offer = {
    //         sdp: offerDescription.sdp,
    //         type: offerDescription.type,
    //     };

    //     // Firebaseに追加
    //     await callDoc.set({offer});
    //     await callDoc.update({ name: this.$route.query.name });

    //   // Remote answer
    //     callDoc.onSnapshot((snapshot) => {
    //         const data = snapshot.data();
    //         if (!this.pc.currentRemoteDescription && data?.answer) {
    //             const answerDescription = new RTCSessionDescription(data.answer);
    //             this.pc.setRemoteDescription(answerDescription);
    //         }
    //     });

    //     // answer 
    //     answerCandidates.onSnapshot((snapshot) => {
    //         snapshot.docChanges().forEach((change) => {
    //             if (change.type === "added") {
    //                 const candidate = new RTCIceCandidate(change.doc.data());
    //                 this.pc.addIceCandidate(candidate);
    //             }
    //         });
    //     });
    },
    
    async answerBtn(val) {
        // const callDoc = this.db.collection("calls").doc(val);
        // const answerCandidates = callDoc.collection("answerCandidates");
        // const offerCandidates = callDoc.collection("offerCandidates");

        // this.pc.onicecandidate = (event) => {
        //     event.candidate && answerCandidates.add(event.candidate.toJSON());
        // };

        // const callData = (await callDoc.get()).data();

        // const offerDescription = callData.offer;
        // await this.pc.setRemoteDescription(
        //     new RTCSessionDescription(offerDescription)
        // );

        // const answerDescription = await this.pc.createAnswer();
        // await this.pc.setLocalDescription(answerDescription);

        // const answer = {
        //     type: answerDescription.type,
        //     sdp: answerDescription.sdp,
        // };

        // await callDoc.update({ answer });

        // offerCandidates.onSnapshot((snapshot) => {
        //     snapshot.docChanges().forEach((change) => {
        //         if (change.type === "added") {
        //         let data = change.doc.data();
        //         this.pc.addIceCandidate(new RTCIceCandidate(data));
        //         }
        //     });
        // });
    }
  }
}
</script>

<style>
button {
  background-color: #008CBA; /* Green */
  color: white;
  padding: 15px 32px;
  text-align: center;
  text-decoration: none;
  display: inline-block;
  font-size: 14px;
  margin: 3px;
}
.disabled{
    display: none;
    background-color: darkgray;
}
</style>