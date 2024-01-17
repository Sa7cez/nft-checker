<script lang="ts">
  import axios from 'axios'
  
  let server = 'https://api-polygon.reservoir.tools'
  let addresses : string = '0x7DBB717cEa1D178e455E650c824b76817cD2041B\n0xA77851ea917A923eD495bD704707c5E274161dE0\n0x9a0BC9B906c4F74C290e3F5Ba4F4254fAe1A7E76'
  let collection = '0x89acaf092909d5e20a18fd6b4a866e3674495b9e'

  let checked = ''
  let empty = ''
  let total = 0
  let stats:any = {}

  const sleep = async (time: number) => new Promise((resolve) => setTimeout(() => resolve(true), time))

  const check = async () => {
    checked = ''
    empty = ''
    total = 0
    const getUserTokens = async (address: string, continuation = null, tokens = []) : Promise<any> =>
      axios.get(`${server}/users/${address}/tokens/v9?collection=${collection}&limit=200${continuation ? `&continuation=${continuation}` : ''}`)
        .then(async ({ data }: any) => {
          if (data.tokens.length > 0) {
            tokens = tokens.concat(data.tokens)
            if (data.continuation)
              return getUserTokens(address, data.continuation, tokens)

          } else {
            empty += `${address}\n`
            console.log(address, `doesn't have NFT`)
            addresses = forCheck.join('\n')
          }
          return tokens
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
      const tokens = await getUserTokens(address as string)

      if (tokens.length > 0)
        checked += `${address}: ${tokens.length} NFTs : ${tokens.map((i: any) => {
          if (!stats[i.token.name]) stats[i.token.name] = 1
          else stats[i.token.name]++
          return `${i.token.tokenId} ${i.token.name}`
        }).join(', ')}\n`
      else
        empty += `${address}\n`

      total += tokens.length
    }
  }
  

</script>

<main>
  <div>
    <label><a href="https://docs.reservoir.tools/reference/supported-chains" target="_blank">Server:</a></label>
    <input bind:value={server} placeholder="Reservoir server"/>
  </div>

  <div>
    <label>Collection address/name:</label>
    <input bind:value={collection} placeholder="Collection address"/>
  </div>
  
  <div>
    <label>Addresses:</label>
    <textarea placeholder="Your wallets here...." bind:value={addresses}  rows="10"></textarea>
  </div>

  <div>
    <button on:click={check}>NFT Token & Name Check!</button>
  </div>

  {#if checked.length > 0}
  <div>
    <label>With NFT:</label>
    <textarea placeholder="Results here" rows="10">{checked}</textarea>
  </div>
  {/if}

  {#if empty.length > 0}
  <div>
    <label>Without NFT:</label>
    <textarea placeholder="Results here" rows="3">{empty}</textarea>
  </div>
  {/if}

  {#if total >0}
  <div>
    <p>Total NFTs count: {total}</p>
    <p>{#each Object.entries(stats) as [key, value]}
      {key}: {value}<br>
    {/each}
    </p>
  </div>
  {/if}

</main>

<style>
  input,
  textarea,
  button {
    width: 100%;
    margin: 0.5rem auto 1rem;
  }
  label {
    display: block;
    text-align: left;
  }
</style>