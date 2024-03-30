<script>
    import 'vidstack/bundle';
    export default {
        data() {
            return {
                transcriptionStatus: 'Not started',
                transcript: '',
            };
        },
        
        mounted() {
            //this.startTranscription();
        },
        
        methods: {
            async makeFetchRequest(url, options) {
                const response = await fetch(url, options);
                return response.json();
            },
            
            async pollForResult(resultUrl, headers) {
                while (true) {
                    console.log("Polling for results...");
                    this.transcriptionStatus = 'Polling for results...';
                    const pollResponse = await this.makeFetchRequest(resultUrl, { headers });

                    if (pollResponse.status === "done") {
                        console.log("- Transcription done:", pollResponse);
                        this.transcriptionStatus = 'Transcription done';
                        this.transcript = pollResponse.result.transcription.full_transcript;
                        break;
                    } else {
                        console.log("Transcription status : ", pollResponse.status);
                        this.transcriptionStatus = `Transcription status: ${pollResponse.status}`;
                        await new Promise(resolve => setTimeout(resolve, 3000));
                    }
                }
            },
            
            async startTranscription() {
                const gladiaKey = "aa699c9b-f25d-4720-9649-ffdd5106aec9";
                const requestData = {
                    audio_url: "https://media-files.vidstack.io/sprite-fight/audio.mp3",
            };
            const gladiaUrl = "https://api.gladia.io/v2/transcription/";
            const headers = {
                "x-gladia-key": gladiaKey,
                "Content-Type": "application/json",
            };

            console.log("- Sending initial request to Gladia API...");
            this.transcriptionStatus = 'Sending initial request to Gladia API...';
            const initialResponse = await this.makeFetchRequest(gladiaUrl, {
                method: "POST",
                headers,
                body: JSON.stringify(requestData),
            });
            console.log("Initial response with Transcription ID :", initialResponse);
            this.transcriptionStatus = `Initial response received: ${JSON.stringify(initialResponse)}`;
                if (initialResponse.result_url) {
                    await this.pollForResult(initialResponse.result_url, headers);
                }
            }
        }
    };
</script>

<template>
    <media-player title="Sprite Fight" src="https://files.vidstack.io/sprite-fight/audio.mp3">
        <media-provider></media-provider>
        <media-audio-layout></media-audio-layout>
    </media-player>

    <div>
        <p>Transcription Status: {{ transcriptionStatus }}</p>
        <p v-if="transcript">Transcript: {{ transcript }}</p>
    </div>
</template>