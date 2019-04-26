# kubectl ns

A [kubectl plugin](https://kubernetes.io/docs/tasks/extend-kubectl/kubectl-plugins/) for interactively switching the current namespace:

![Demo](img/demo.gif)

This makes it easier to work with different namespaces in the same cluster.

Also see [kubectl-ctx](https://github.com/weibeld/kubectl-ctx) for switching between contexts (e.g. clusters).

## Installation

1. Download the `kubectl-ns` script:

    ~~~bash
    curl -O https://raw.githubusercontent.com/weibeld/kubectl-ns/master/kubectl-ns
    ~~~

2. Make it executable:

    ~~~bash
    chmod +x kubectl-ns
    ~~~
    
3. Move it to *any* directory in your `PATH` (you might crate a `~/.kubectl-plugins` folder for all your kubectl plugins and add it to your `PATH`):

    ~~~bash
    mv kubectl-ns ~/.kubectl-plugins
    ~~~~

Now, you can verify that the plugin is correctly installed by running the following command and checking that the `kubectl-ns` script is in the output:

~~~bash
kubectl plugin list
~~~~

To uninstall the plugin, simply delete the `kubectl-ns` script.

## Dependencies

The plugin depends on the [fzf](https://github.com/junegunn/fzf) command being available on your system.

You can install fzf as follows:

- Homebrew (macOS) and Linuxbrew (Linux):
    ~~~bash
    brew install fzf
    ~~~
- From source (macOS and Linux):
    ~~~bash
    git clone https://github.com/junegunn/fzf.git ~/.fzf
    ~/.fzf/install
    ~~~
- For further installation options, see [here](https://github.com/junegunn/fzf#installation)

## Usage

Interactively change the current namespace:

~~~bash
kubectl ns
~~~

List all namespaces:

~~~bash
kubectl ns -l
~~~

Update the namespaces cache:

~~~bash
kubectl ns -u
~~~

### Cache

The cache feature speeds up the `kubectl ns` and `kubectl ns -l` commands. This is because these commands need to retrieve the full list of namespaces, which is usually only stored in the cluster and thus needs to be retrieved from the API server. Without the cache, these commands would take as long as it takes you to execute `kubectl get namespaces`. With the cache, they are blazingly fast, as they get the full list of namespaces from the local cache rather than from the API server.

The cache is created automatically when running `kubectl ns` or `kubectl ns -l` for the first time with a given cluster (cache location is `~/.kube/kubectl-ns`). You can manually update the cache at any time by running `kubectl ns -u` (there are no automatic updates of the cache, so you should run `kubectl ns -u` whenever you create or delete namespaces).

## How It Works

When you run `kubectl ns`, the plugin changes your local [*kubeconfig*](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/) configuration as shown below:

![How it works](img/how-it-works.png)

As you can see, the plugin changes the **namespace** element of the **current context** in your *kubeconfig* configuration (the plugin always works on the current context).

This means that the next time you use `kubectl`, it will use the new namespace for the same cluster.

Note that the `ns` command physically changes one of your *kubeconfig* files (the default *kubeconfig* file is `~/.kube/config`, but you can have multiple *kubeconfig* files by by listing them in the [`KUBECONFIG`](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/#the-kubeconfig-environment-variable) environment variable).

The `ns -l` command displays the namespaces of the **cluster** which is referenced by your current context.
