## Debug Kubernetes Applications

Your Kubernetes application is running well, and then all of a sudden the service stops responding. How do you debug? You created a deployment but its not coming up. Is your pod status shown as pending? How do you debug deployments and pods, get their logs, see the filesystem layout? Horizontal Pod Autoscaler is not scaling pods. Is your cluster running out of capacity? Or are the metrics not available? Having DNS lookup failures for services? Is your PVC status shown pending? Is kubectl not able to find nodes? This repo will share different ways your applications on k8s crash and burn, and more importantly to recover from them.

## How to contribute

1. Clone this repository 

   ```
   git clone https://github.com/aws-samples/debug-k8s-apps.git
   ```

1. Install [Hugo](https://gohugo.io/getting-started/installing/) in your development environment
1. Setup Hugo:

   ```
   cd debug-k8s-apps
   git submodule init ; git submodule update
   ```

1. Make changes to the presentation in `content/\_index.md`
1. After making changes, generate the HTML content by running `hugo server`
1. Commit and ship!

The presentation site is accesible at https://aws-samples.github.io/debug-k8s-apps/ 

## License

This library is licensed under the MIT-0 License. See the LICENSE file.

