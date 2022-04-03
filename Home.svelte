<script>
  import { onMount } from "svelte";

  import axios from "axios";

  const options = {
    method: "GET",
    url: "https://data-imdb1.p.rapidapi.com/titles/search/title/",
    params: { info: "mini_info", limit: "10", page: "1", titleType: "movie" },
    headers: {
      "X-RapidAPI-Host": "data-imdb1.p.rapidapi.com",
      "X-RapidAPI-Key": "36c4bcc195msh0c3bf45a11ace6cp1ef288jsnf6e9d8b6d2c0"
    }
  };

  let searchResult = "";
  let error = "";

  onMount(() => {
    const submitButton = document.querySelector("#submit");
    const input = document.querySelector("#input");
    const errorSpan = document.querySelector("#error");
    const resultsContainer = document.querySelector("#results");
  });

  const disableUi = () => {
    input.disabled = true;
    //submitButton.disabled = true;
  };

  const enableUi = () => {
    input.disabled = false;
    //submitButton.disabled = false;
  };

  const clearPreviousResults = () => {
    searchResult = "";
    error = "";
  };

  const isInputEmpty = input => {
    if (!input || input === "") return true;
    return false;
  };

  const showError = er => {
    error = `🚨 ${er} 🚨`;
  };

  const showResults = results => {
    results.forEach(result => {
      searchResult += `<div class="results__item">
                                                      <a href="https://en.wikipedia.org/?curid=${
                                                        result.pageId
                                                      }" target="_blank" class="card animated bounceInUp">
                                                          <h2 class="results__item__title">${
                                                            result.title
                                                          }</h2>
                                                          <p class="results__item__intro">${
                                                            result.intro
                                                          }</p>
                                                      </a>
                                                  </div>`;
    });
  };

  const gatherData = pages => {
    // console.table(pages);
    // const results = Object.values(pages).map(page => ({
    //   pageId: page.pageid,
    //   title: page.title,
    //   intro: page.extract
    // }));
    // showResults(results);
  };

  const getData = async () => {
    const userInput = input.value;
    if (isInputEmpty(userInput)) return;

    options.url += userInput;
    console.log("url", options.url);
    clearPreviousResults();
    disableUi();

    try {
      axios
        .request(options)
        .then(function(response) {
          console.log(response.data);
        })
        .catch(function(error) {
          throw new Error(data.error.info);
          console.error(error);
        });

      gatherData(response.data);
    } catch (error) {
      showError(error);
    } finally {
      enableUi();
    }
  };

  const getAutocomplete = async () => {
    const userInput = input.value;

    params.gpssearch = userInput;
    clearPreviousResults();
    //disableUi();

    try {
      const { data } = await axios.get(endpoint, { params });

      if (data.error) throw new Error(data.error.info);
      gatherData(data.query.pages);
    } catch (error) {
      showError(error);
    } finally {
      enableUi();
    }
  };

  function onInput(e) {
    let inptTxt = e.currentTarget.value;
    // if (inptTxt.length > 0) {
    //   setTimeout(() => {
    //     getAutocomplete();
    //   }, 500);
    // }
    if (e.key === "Enter") {
      getData();
    }
  }
</script>

<style>
</style>

<header>
  <h1 class="header__title">imdb.com</h1>
  <div class="header__search-bar">
    <input
      id="input"
      aria-label="What do you want to know?"
      type="text"
      class="search-bar__input"
      placeholder="What do you want to know?"
      on:keyup={onInput}
      autofocus
      required
    />
    <button
      id="submit"
      type="button"
      class="search-bar__button"
      aria-label="search"
      on:click={getData}
    >
      <svg
        version="1.1"
        id="Layer_1"
        x="0px"
        y="0px"
        width="15px"
        height="15px"
        viewBox="0 0 25 25"
        enableBackground="new 0 0 25 25"
      >
        <path
          fill="#C0C0C0"
          d="M23.888,21.266l-5.629-5.628c1.1-1.604,1.742-3.545,1.742-5.637C20.001,4.478,15.526,0,10,0
                        C4.478,0,0,4.478,0,10.001S4.478,20,10,20c2.204,0,4.233-0.721,5.885-1.927l5.598,5.597c0.717,0.718,1.838,0.761,2.5,0.097
                        C24.647,23.104,24.604,21.983,23.888,21.266z M10,16.748c-3.728,0-6.749-3.021-6.749-6.747c0-3.728,3.021-6.749,6.749-6.749c3.729,0,6.749,3.021,6.749,6.749C16.749,13.728,13.729,16.748,10,16.748z"
        />
      </svg>
    </button>
  </div>
</header>
<div class="results">
  <p id="error" class="results__error">{@html error}</p>
  <div id="results" class="results__items">{@html searchResult}</div>
</div>