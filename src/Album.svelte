<script>
import moment from 'moment';
import Suspense from 'svelte-suspense';
// import Suspense from './Suspense.svelte';

export let album;
$: image=album["im:image"][2].label
$: title=album.title.label
$: link=album.id.label
$: price=album["im:price"].label
$: date=album["im:releaseDate"].label

function formatDate(date, format) {
  return moment(date).format(format);
};

// index in events array
const DONE = 1

</script>

<ul class="album">
    <li class="album-item">
        <Suspense let:intents={{done, failed}} timeout=0>
            <div slot="fallback" class="album-img">
                <img alt="" src="https://media.giphy.com/media/y1ZBcOGOOtlpC/200.gif" />
            </div>
            <a href={link} target="blank" class="link">
                <img class="album-img"
                     on:load="{() => done(void 0)}"
                     src={image}
                     alt={'itunes' + Math.random()} />
            </a>
        </Suspense>
    </li>
    <li class="title album-item">
        <a href={link} target="blank" class="link">
            {title.slice(0, 20)}..</a></li>
    <li class="price album-item">Price:{price}</li>
    <li class="date album-item">Released:{formatDate(date, "MMM Do YY")}</li>
</ul>
