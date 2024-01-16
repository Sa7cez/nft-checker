<script lang="ts">
  import svelteLogo from './assets/svelte.svg'
  import viteLogo from '/vite.svg'
  import Counter from './lib/Counter.svelte'
  import axios from 'axios'
  
  let server = 'https://api-polygon.reservoir.tools'
  let addresses : string = '0x7DBB717cEa1D178e455E650c824b76817cD2041B\n0xA77851ea917A923eD495bD704707c5E274161dE0\n0x9a0BC9B906c4F74C290e3F5Ba4F4254fAe1A7E76'
  let collection = '0x89acaf092909d5e20a18fd6b4a866e3674495b9e'

  let checked : string[] = []

  const sleep = async (time) => new Promise((resolve) => setTimeout(() => resolve(true), time))

  const check = async () => {
    const getUserTokens = async (address: Hex) =>
      axios.get(`${server}/users/${address}/tokens/v9?collection=${collection}`)
        .then(({ data }: any) => {
          if (data.tokens.length > 0) {
            checked += `${address} ${data.tokens.map((i: any) => `${i.token.tokenId} ${i.token.name}`)}\n`
          } else {
            console.log(address, `doesn't have NFT`)
            addresses = forCheck.join('\n')
          }
        })
        .catch(async (err: any) => {
          console.error(err.message)
          await sleep(2500)
          return getUserTokens(address)
        })

    const forCheck = addresses.split(/\n|,/)
    console.log(forCheck)
    while (forCheck.length > 0) {
      const address = forCheck.pop()
      const tokens = await getUserTokens(address)
    }
  }
  

</script>

<main>
  <div>
    <label>Server:</label>
    <input bind:value={server} placeholder="Reservoir server"/>
  </div>

  <div>
    <label>Collection contract/name:</label>
    <input bind:value={collection} placeholder="Collection address"/>
  </div>
  
  <div>
    <label>Addresses:</label>
    <textarea placeholder="Your wallets here...." bind:value={addresses}  rows="10"></textarea>
  </div>

  <div class="card">
    <button on:click={check}>NFT Token & Name Check!</button>
  </div>

  <div>
    <textarea placeholder="Results here" rows="10">{checked}</textarea>
  </div>

</main>

<style>
  input,
  textarea {
    width: 100%;
    margin: 0.5rem;
  }
  .card {
    margin: 0.5rem;
  }
  button {
    margin: 0;
  }
</style>