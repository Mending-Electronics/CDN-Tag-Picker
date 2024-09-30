const app = Vue.createApp({

    data() {
      return {
        isGithubTableHidden: true,
        isJsDelivrTableHidden: true,
        isCdnjsTableHidden: true,

        libraries: window.libraries
      };
    },



    mounted() {
      this.libraries.forEach(lib => {

        // CDN JS API Request
        axios.get(lib.cdnjs.replace("cdnjs.com","api.cdnjs.com")).then(response => {
          lib.cdnAssets = response.data.assets[0].sri;
          lib.cdnjsLastestVersion = response.data.version;
        });
     

        // jsDelivr API Request
        axios.get(lib.jsdelivr.replace("www.jsdelivr.com","data.jsdelivr.com/v1")).then(response => {
          lib.jsdelivrLastestVersion = response.data.tags.latest; // Get the first version
          // console.log(lib.jsdelivrLastestVersion);

          if (lib.jsdelivrLastestVersion !== "") {
            const jsdelivrLibVersionAPI = lib.jsdelivr.replace("www.jsdelivr.com","data.jsdelivr.com/v1") + "@" + lib.jsdelivrLastestVersion;
            // console.log(jsdelivrLibVersionAPI);

            axios.get(jsdelivrLibVersionAPI).then(response => {
              lib.jsdelivrAssets = response.data.files[0].files; // Corrected the response structure .files
              lib.jsdelivrAssetsDirectory = response.data.files[0].name
              console.log(lib.name);
              console.log(lib.jsdelivrAssetsDirectory);
              console.log(lib.jsdelivrAssets);
              console.log("parent");
              console.log(response.data);


            }).catch(error => {
              console.error('Error fetching jsDelivr assets:', error);
            });
          }
          
        }).catch(error => {
          console.error('Error fetching version:', error);
        });



         // Github API Request
        const githubReleasesAPI = lib.githubRepo.replace("github.com","api.github.com/repos") + '/releases';
        const githubIssuesAPI = lib.githubRepo.replace("github.com","api.github.com/repos") + '/issues';

        axios.get(githubReleasesAPI).then(response => {
          lib.githubAssets = response.data;
          // console.log(lib.githubAssets);

          lib.latestGithubRelease = lib.githubAssets.find(release => !release.draft && !release.prerelease);


          if (lib.latestGithubRelease) {
            lib.latestGithubReleaseVersion = lib.latestGithubRelease.tag_name.replace("v", "");
            // console.log(`Cleaned Github tag name: ${lib.latestGithubReleaseVersion}`);
          } else {
            console.log("No suitable release found.");
          }
        }).catch(error => {
          // console.error("Error fetching releases:", error);
		  
		  
          if (error.response && error.response.data && error.response.data.message) {
          const errorMessage = error.response.data.message;
			if (errorMessage.startsWith('API rate limit exceeded')) {
			  lib.githubMessageAPI = "API rate limit exceeded";
			  // console.log(lib.githubMessageAPI);
			}
		  }
		  
		  
		  
        });



      });
    },


    methods: {

      copyToClipboard(text) {
        navigator.clipboard.writeText(text).then(() => {
          alert('Copied to clipboard');
        }).catch(err => {
          console.error('Failed to copy: ', err);
        });
      },

      toggleGithubTable() {
        this.isGithubTableHidden = !this.isGithubTableHidden;
      },
      toggleJsDelivrTable() {
        this.isJsDelivrTableHidden = !this.isJsDelivrTableHidden;
      },
      toggleCdnjsTable() {
        this.isCdnjsTableHidden = !this.isCdnjsTableHidden;
      },



      downloadcdnContent(url, filename) {
        axios({
          url: url,
          method: 'GET',
          responseType: 'blob'
        })
          .then((response) => {
            if (response.status === 200) {
              const contentUrl = window.URL.createObjectURL(new Blob([response.data]));
              const link = document.createElement('a');
              link.href = contentUrl;
              link.setAttribute('download', filename);
              document.body.appendChild(link);
              link.click();
              document.body.removeChild(link);
              window.URL.revokeObjectURL(contentUrl);
            } else {
              console.error('Error: Received non-200 status code');
            }
          })
          .catch((error) => {
            console.error('Error downloading the content:', error);
          });
      }

    }


});

app.mount('#app');