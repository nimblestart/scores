<script lang="ts">
    import { userBetarenaSettings } from '$lib/store/user-settings';
</script>

<svg width="80" height="80" viewBox="0 0 80 80" fill="none" xmlns="http://www.w3.org/2000/svg">
    <rect width="80" height="80" rx="4" style={$userBetarenaSettings.theme == 'Dark' ? 'fill: #616161;' : 'fill: #F2F2F2;'} />
    <path
        d="M37.2006 29.653C38.1184 29.4382 39.0581 29.3308 40.0007 29.333C49.334 29.333 54.6673 39.9997 54.6673 39.9997C53.858 41.5139 52.8927 42.9394 51.7873 44.253M42.8273 42.8264C42.4611 43.2194 42.0195 43.5346 41.5289 43.7532C41.0382 43.9718 40.5085 44.0894 39.9714 44.0989C39.4344 44.1083 38.9009 44.0095 38.4028 43.8084C37.9047 43.6072 37.4523 43.3077 37.0724 42.9279C36.6926 42.5481 36.3932 42.0956 36.192 41.5976C35.9908 41.0995 35.892 40.566 35.9015 40.0289C35.911 39.4918 36.0285 38.9622 36.2472 38.4715C36.4658 37.9808 36.781 37.5392 37.174 37.173M47.9207 47.9197C45.6414 49.657 42.8661 50.6195 40.0007 50.6664C30.6673 50.6664 25.334 39.9997 25.334 39.9997C26.9925 36.9089 29.2928 34.2085 32.0807 32.0797L47.9207 47.9197Z"
        style={$userBetarenaSettings.theme == 'Dark' ? 'stroke: #A8A8A8;' : 'stroke: #8C8C8C;'} stroke-width="2" stroke-linecap="round" stroke-linejoin="round" />
    <path d="M25.334 25.333L54.6673 54.6663" style={$userBetarenaSettings.theme == 'Dark' ? 'stroke: #A8A8A8;' : 'stroke: #8C8C8C;'} stroke-width="2" stroke-linecap="round"
        stroke-linejoin="round" />
</svg>