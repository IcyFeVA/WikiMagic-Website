<script>
  import { onMount } from "svelte";

  const axios = require("axios").default;

  const endpoint = "https://en.wikipedia.org/w/api.php?";
  const params = {
    origin: "*",
    format: "json",
    action: "query",
    prop: "pageprops|pageimages|description",
    generator: "prefixsearch",
    gsrlimit: 4,
    ppprop: "displaytitle",
    piprop: "thumbnail",
    pilimit: 6,
    gpsnamespace: 0,
    gpslimit: 6
  };
  /*const params = {
      origin: "*",
      format: "json",
      action: "query",
      prop: "extracts",
      exchars: 250,
      exintro: true,
      explaintext: true,
      generator: "search",
      gsrlimit: 20
    };*/

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
    console.table(pages);
    const results = Object.values(pages).map(page => ({
      pageId: page.pageid,
      title: page.title,
      intro: page.extract
    }));

    showResults(results);
  };

  const getData = async () => {
    const userInput = input.value;
    if (isInputEmpty(userInput)) return;

    params.gsrsearch = userInput;
    clearPreviousResults();
    disableUi();

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
    if (inptTxt.length > 0) {
      setTimeout(() => {
        getAutocomplete();
      }, 500);
    }
    if (e.key === "Enter") {
      getData();
    }
  }
</script>

<style>
  button {
    background: #ff3e00;
    color: white;
    border: none;
    padding: 8px 12px;
    border-radius: 2px;
  }

  .header__image {
    max-width: 300px;
    width: 100%;
  }

  .header__title {
    font-family: "Zilla Slab", serif;
    font-weight: 500;
    font-size: 2.5rem;
  }

  .header__search-bar {
    display: flex;
    justify-content: space-between;
    width: 320px;
    padding: 5px;
    border: 1px solid #f4f4f4;
    margin: 40px auto;
    box-shadow: 0px 2px 4px #dfdfdf;
  }

  .header__search-bar:hover {
    box-shadow: 0px 3px 5px #c0c0c0;
  }

  .search-bar__input {
    font-size: 1rem;
    width: 85%;
    border: none;
    padding: 0.5rem;
    outline-color: #1c1b1b;
  }

  .search-bar__button {
    padding: 0.75rem;
    border: none;
    outline-color: #1c1b1b;
    background-color: #fcfcfc;
    cursor: pointer;
  }
  .search-bar__input:disabled,
  .search-bar__button:disabled {
    cursor: not-allowed;
  }

  .results {
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
  }

  .results__items {
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
  }

  .results__error {
    color: #ef0000;
  }

  .results__item {
    background: #fff;
    border-radius: 2px;
    display: inline-block;
    height: 275px;
    margin: 1rem;
    /* position: relative; */
    width: 300px;
    box-shadow: 0 1px 3px rgba(0, 0, 0, 0.12), 0 1px 2px rgba(0, 0, 0, 0.24);
    transition: all 0.3s cubic-bezier(0.25, 0.8, 0.25, 1);
    cursor: pointer;
    overflow: hidden;
  }
  .results__item:hover {
    box-shadow: 0 3px 6px rgba(0, 0, 0, 0.16), 0 3px 6px rgba(0, 0, 0, 0.23);
  }
  .results__item__title {
    font-family: "Zilla Slab", serif;
    font-weight: 500;
    margin: 20px;
  }
  .results__item__intro {
    margin: 0 20px;
    padding-bottom: 20px;
  }
</style>

<header>
  <img
    src="https://upload.wikimedia.org/wikipedia/commons/d/de/Wikipedia-logo_%28inverse%29.png"
    alt="Wikipedia Logo"
    width="300"
    class="header__image"
  />
  <h1 class="header__title">Wikipedia.org</h1>
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