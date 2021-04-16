<template>
    <div class="player" :class="{ playing }">
        <div class="disc">
            <div class="disc-cover"></div>
            <div class="disc-bottom">
                <a href class="playBtn btn" @click.prevent="playOrPause">
                    <i v-if="playing" class="material-icons">pause</i>
                    <i v-else class="material-icons">play_arrow</i>
                </a>
                <div class="disc-antenna"></div>
            </div>
        </div>

        <div class="switch">
            <a href class="nextBtn btn" :class="{ disabled: track === songs.length - 1 }" @click.prevent="switchTrack(1)">
                <i class="material-icons">skip_next</i>
            </a>
            <a href class="prevBtn btn" :class="{ disabled: track === 0 }" @click.prevent="switchTrack(-1)">
                <i class="material-icons">skip_previous</i>
            </a>
        </div>

        <div class="controller">
            <input type="checkbox" id="shuffle" v-model="shuffle">
            <label for="shuffle" class="shuffleBtn btn">
                <i class="material-icons">shuffle</i>
            </label>

            <div class="row">
                <div class="songName">
                    <strong>{{songs[track].songName}}</strong>
                    <p>{{songs[track].singer}}</p>
                </div>
                <div class="volume">
                    <div class="bar js-bar" data-type="volume">
                        <div class="bar-value" :style="{ left:`${volume * 100}%` }"></div>
                    </div>
                    <i class="volume-icon material-icons">volume_up</i>
                </div>
            </div>

            <div class="progress">
                <p>{{displayTime(currentTime)}}</p>
                <div class="bar js-bar" data-type="progress">
                    <div class="bar-value" :style="{ left:`${progress}%` }"></div>
                </div>
                <p>{{displayTime(duration)}}</p>
            </div>
        </div>

        <audio :src="`${publicPath}mp3/${songs[track].fileName}.mp3`" ref="audio"></audio>
    </div>
</template>

<script>
import { setTimeout } from 'timers';
export default {
    name: 'Player',
    data() {
        return {
            publicPath: process.env.BASE_URL,
            isMouseDown: false,
            shuffle: false,
            playing: false,
            duration: 0,
            progress: 0,
            currentTime: 0,
            volume: 0,
            track: 0,
            songs: [
                {
                    fileName: 'The_Long_Night',
                    songName: '1. The Long Night',
                    singer: 'Jay'
                },
                {
                    fileName: 'The_End',
                    songName: '2. The End',
                    singer: 'Jolin'
                },
                {
                    fileName: 'Central_Park',
                    songName: '3. Central Park End',
                    singer: 'Andy'
                },
            ],
        }
    },
    watch: {
        progress(val) {
            if (val < 100) return;

            if (this.shuffle) {
                // 取得非當前播放的隨機曲目
                let currentTrack = this.track;
                while(currentTrack === this.setRandomTrack()) {
                    this.setRandomTrack();
                }
            }
            else if (this.track < this.songs.length - 1) {
                this.track++;
            }
        }
    },
    methods: {
        playOrPause() {
            if (this.playing) {
                this.$refs.audio.pause();
            }
            else {
                this.$refs.audio.play();
            }
        },
        displayTime(time) {
            let t = Math.floor(time);
            let min = Math.floor(t / 60);
            let sec = t % 60;
            return `${min}:${sec < 10 ? 0 : ''}${sec}`;
        },
        barController(e) {
            if (e.type === 'mousemove' && !this.isMouseDown) return;

            let pos = (e.offsetX / e.currentTarget.offsetWidth * 100).toFixed(2);
            e.currentTarget.firstChild.style.left = `${pos}%`;

            switch (e.currentTarget.dataset.type) {
                case 'progress':
                    let currentTime = e.offsetX / e.currentTarget.offsetWidth * this.duration;
                    this.$refs.audio.currentTime = currentTime < 0 ? 0 : currentTime;
                    break;
                case 'volume':
                    let valume = e.offsetX / e.currentTarget.offsetWidth;
                    this.$refs.audio.volume = valume < 0 ? 0 : valume;
                    break;
                default:
                    break;
            }
        },
        switchTrack(val) {
            this.track += val;
            if (!this.playing) {
                this.progress = 0;
                this.currentTime = 0;
            }
        },
        setRandomTrack() {
            this.track = Math.floor(Math.random() * this.songs.length);
            return this.track;
        },
    },
    mounted() {
        this.$refs.audio.addEventListener('play', () => this.playing = true);
        this.$refs.audio.addEventListener('pause', () => this.playing = false);
        this.$refs.audio.addEventListener('loadedmetadata', e => {
            this.duration = e.target.duration;
            this.volume = e.target.volume;
            if (this.playing) {
                this.$refs.audio.play();
            }
        });
        this.$refs.audio.addEventListener('timeupdate', e => {
            let audio = e.target;
            if (!audio.duration) return;
            this.progress = parseFloat((audio.currentTime / audio.duration * 100).toFixed(2));
            this.currentTime = audio.currentTime;
        });

        document.addEventListener('mousedown', () => this.isMouseDown = true);
        document.addEventListener('mouseup', () => this.isMouseDown = false);

        document.querySelectorAll('.js-bar').forEach(el => {
            el.addEventListener('mousedown', this.barController);
        });
        document.querySelectorAll('.js-bar').forEach(el => {
            el.addEventListener('mousemove', this.barController);
        });
    },
}
</script>

<style lang="scss" src="@/assets/sass/style.scss"></style>