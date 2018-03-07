<template>
    <yd-layout title="ScrollNav">
        <yd-indexlist :dataArray="datalist" indexname="name">
            
        </yd-indexlist>
    </yd-layout>
</template>

<script>
export default {
    data() {
        return {
            datalist: [
                { name: '爱你' },
                { name: '##' },
                { name: 'A' },
                { name: 'B' },
                { name: 'AA' },
                { name: 'C' },
                { name: 'D' },
                { name: 'E' },
                { name: 'AAA' },
                { name: 'F' },
                { name: '1' },
                { name: 'G' },
                { name: 'GG' },
                { name: 'H' },
                { name: 'I' },
                { name: 'J' }
            ]
        }
    }
}
</script>

<style>

</style>
