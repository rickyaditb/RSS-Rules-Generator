<script>
  import { onMount } from "svelte";
  import Parser from "rss-parser";
  import axios from "axios";
  let parser = new Parser();
  let mal = [];
  let rss = [];
  let download = [];
  let pagination = {};
  let selected;
  let result;

  (async () => {
    let cors_proxy = "https://cors.rickyaditb.my.id/";
    let feed = await parser.parseURL(
      cors_proxy + "https://nyaa.si/?f=0&c=0_0&q=subsplease+1080p&page=rss"
    );

    feed.items.forEach((item) => {
      rss.push(item.title);
    });
    rss = rss;
  })();

  axios
    .get("https://api.jikan.moe/v4/seasons/now?page=1")
    .then(function (response) {
      mal = response.data.data;
      pagination = response.data.pagination;
    });

  const nextPage = (page) => {
    axios
      .get(`https://api.jikan.moe/v4/seasons/now?page=${page + 1}`)
      .then(function (response) {
        mal = response.data.data;
        pagination = response.data.pagination;
      });
  };

  const prevPage = (page) => {
    axios
      .get(`https://api.jikan.moe/v4/seasons/now?page=${page - 1}`)
      .then(function (response) {
        mal = response.data.data;
        pagination = response.data.pagination;
      });
  };

  const searchAnime = (title) => {
    result = [];
    selected = undefined;
    let query = title.split(" ").slice(0, 2).join(" ");
    rss.forEach((item) => {
      if (item.toLowerCase().includes(query.toLowerCase())) {
        result.push(item);
        selected = query;
      }
    });
  };

  const searchAnime2 = (title) => {
    result = [];
    selected = undefined;
    rss.forEach((item) => {
      if (item.toLowerCase().includes(title.toLowerCase())) {
        result.push(item);
        selected = title;
      }
    });
  };

  const addToDownload = () => {
    if (selected) {
      download.push(selected);
      download = download;
    }
  };

  const exportRules = () => {
    let rules = {};
    download.forEach((item) => {
      let template = {
        [item]: {
          addPaused: null,
          affectedFeeds: ["https://subsplease.org/rss/?t&r=1080"],
          assignedCategory: "",
          enabled: true,
          episodeFilter: "",
          ignoreDays: 0,
          lastMatch: "",
          mustContain: item,
          mustNotContain: "",
          previouslyMatchedEpisodes: [],
          savePath: "",
          smartFilter: false,
          torrentContentLayout: null,
          useRegex: false,
        },
      };
      rules[item] = template[item];
    });
    console.log(rules);
    const jsonData = JSON.stringify(rules);
    const blob = new Blob([jsonData], { type: "application/json" });
    const url = URL.createObjectURL(blob);
    const link = document.createElement("a");
    link.download = "rss.json";
    link.href = url;
    link.click();
    URL.revokeObjectURL(url);
  };
</script>

<main class="grid grid-cols-3 gap-1">
  <section class="bg-white p-3 rounded shadow h-screen pb-20">
    <p class="text-center mb-5 font-bold text-gray-700 text-2xl">
      All Currently Airing
    </p>
    <div class="flex flex-col gap-3 h-full overflow-scroll">
      {#each mal as anime}
        <button
          class="flex gap-2 hover:bg-green-200 cursor-pointer"
          id={anime.mal_id}
          on:click={() => searchAnime(anime.title)}
        >
          <img src={anime.images.webp.image_url} class="w-12" alt="" />
          <p class="text-left">{anime.title}</p>
        </button>
      {/each}
      <div>
        {#if pagination.current_page !== 1}
          <button
            class="bg-red-500 text-white p-3 font-bold rounded"
            on:click={() => prevPage(pagination.current_page)}>Previous</button
          >
        {/if}
        {#if pagination.has_next_page}
          <button
            class="bg-green-500 text-white px-7 py-3 font-bold rounded"
            on:click={() => nextPage(pagination.current_page)}>Next</button
          >
        {/if}
      </div>
    </div>
  </section>
  <section class="bg-white p-3 rounded shadow h-screen">
    <div class="mb-3">
      <p class="text-center mb-5 font-bold text-gray-700 text-2xl">
        Manual Search
      </p>
      <input
        type="text"
        class="bg-gray-200 w-full p-3 rounded"
        placeholder="Use this if you can't find anything using left selector"
        on:input={(e) => searchAnime2(e.target.value)}
      />
    </div>
    <button
      on:click={addToDownload}
      class="bg-green-500 text-white font-bold p-3 rounded"
      >Add to RSS Downloader</button
    >
    <button on:click={exportRules} class="bg-purple-500 text-white font-bold p-3 rounded">Export RSS Config</button>
  </section>
  <section class="grid grid-rows-3 h-screen gap-1">
    <div class="bg-white p-3 rounded shadow pb-20">
      <p class="text-center mb-5 font-bold text-gray-700 text-2xl">Found RSS</p>
      <div class="h-full overflow-scroll">
        {#if result}
          {#each result as x, index}
            <div>
              <p>{x}</p>
            </div>
          {/each}
        {:else}
          {#each rss as feed}
            <div>
              <p>{feed}</p>
            </div>
          {/each}
        {/if}
      </div>
    </div>
    <div class="bg-white p-3 rounded shadow row-span-2 pb-20">
      <p class="text-center mb-5 font-bold text-gray-700 text-2xl">Added RSS</p>
      <div class="flex flex-col gap-3 overflow-scroll h-full">
        {#each download as item, index}
          <button
            on:click={() => {
              download.splice(index, 1);
              download = download;
            }}
            class="text-left p-3 bg-gray-200 rounded"
          >
            {item}
          </button>
        {/each}
      </div>
    </div>
  </section>
</main>

<style>
</style>
