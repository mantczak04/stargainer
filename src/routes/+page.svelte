<script lang="ts">
    import { Octokit } from "@octokit/rest";
    import { onMount } from "svelte";
    
    const API_KEY = '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';
    const octokit = new Octokit({auth: API_KEY});

    interface Stargazer {
        name: string,
        avatar: string
    }

    const README_OWNER: string = 'mantczak04';

    const owner: string = 'mantczak04';
    const repo: string = 'coolgit';

    const stargazersUrl = `https://api.github.com/repos/${owner}/${repo}/stargazers`;

    let lastPageOfStargazersUrl: string = '';
    let lastStargazers: Stargazer[] = [];
    let newReadmeContent: string = '';

  async function refreshStargazers(){
    try{
      lastPageOfStargazersUrl = await getLastPageOfStargazers(stargazersUrl);
      lastStargazers = await getLastThreeStargazers(lastPageOfStargazersUrl);
      console.log(lastStargazers);
      newReadmeContent = makeReadmeFromStargazers(lastStargazers);
      console.log(newReadmeContent);
      updateReadmeStargazers(newReadmeContent);
    } catch (error) {
      console.error(error);
    }
  }


  function makeReadmeFromStargazers(stargazersArray : Stargazer[]) : string{
    let output: string = '<h1> Thank you for stars! : </h1> \n';
    output+= '<table align="center"> <tr>'
    let secondRow: string = '';
    stargazersArray.forEach(stargazer => {
      output += `<th>	<img src="${stargazer.avatar}" width="200px" title="${stargazer.name}"></th>`;
      secondRow += `<td align="center"><a href="https://github.com/${stargazer.name}">@${stargazer.name} </td>`
    });
    secondRow = '<tr>' + secondRow +'</tr>';
    output+=secondRow;
    output+='</table>';

    return output;
  }

  async function updateReadmeStargazers(newContent: string): Promise<void>{
    try{
      const {data: readme} = await octokit.repos.getContent({
        owner: README_OWNER,
        repo: README_OWNER,
        path: 'README.md'
      });

      if('content' in readme){
        await octokit.repos.createOrUpdateFileContents({
        owner: README_OWNER,
        repo: README_OWNER,
        path: 'README.md',
        message: 'Update new stargazers',
        content: btoa(newContent),
        sha: readme.sha
      });
    } else{
      console.error('README.md is not a file');
    } 
  }
    catch(error){
      console.error(error);
    }
  }


    async function getLastPageOfStargazers(url: string): Promise<string>{
        const response = await fetch(url);
        const headers = response.headers;
        
        let lastPageUrl : string = url;

        let lastPage: string = '';

        const linkHeaderValue = headers.get("link");

        if(!linkHeaderValue){
            return lastPageUrl+"?page=1";
        } else {
            let indexOfLastWord : number = linkHeaderValue!.indexOf("last");
            for(let i: number=indexOfLastWord-12; i<=indexOfLastWord-7; i++){
                if (linkHeaderValue!.charAt(i) >= '0' && linkHeaderValue!.charAt(i) <= '9'){
                    lastPage = lastPage + linkHeaderValue?.charAt(i);
                }
            }
            lastPageUrl = lastPageUrl+"?page="+lastPage;
        }

        return lastPageUrl;
    }


    async function getLastThreeStargazers(url: string): Promise<Stargazer[]> {
  try {
    const response = await fetch(url);

    if (!response.ok) {
      throw new Error(`Error in ${url}: Couldn't get response. Status code: ${response.status}`);
    }

    const jsonDataArray = await response.json();
    const lastThreeStargazers: Stargazer[] = [];

    for (const jsonData of jsonDataArray.slice(-3)) {
      lastThreeStargazers.push({
        name: jsonData['login'],
        avatar: jsonData['avatar_url']
      });
    }

    return lastThreeStargazers;

  } catch (error) {
    console.error('Error fetching stargazers:', error);
    return []; // Or throw error if handling within the caller is preferable
  }
}


</script>


<h1>coolgit - star collector</h1>
<div class="main-container">
    <h1>stargazers:</h1>
    {#each lastStargazers as {name, avatar}}
    <div class="stargazer">
        <img src={avatar} alt="">
        <p>{name}</p>
    </div>
    {/each}

    <button on:click={refreshStargazers}>generate</button>
</div>
