# substrate node with poe implemented


### Navigate to the directory where you want to clone the repository
This would be whatever folder you want on your local computer, for example:
```sh
cd tutorials
```

### Clone the repository with only the latest commit history
```sh
git clone --depth 1 https://github.com/fzeeshan/substrate-tutorials.git
```
### Navigate into the cloned repository directory
```sh
cd substrate-tutorials
```
### Enable sparse-checkout
```sh
git sparse-checkout init
```
# Specify the folder you want to include in the cloned repository
```sh
git sparse-checkout set poe
```
# Fetch the content of the specified folder
```sh
git pull origin main
```
