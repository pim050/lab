 on prodesk01 + remote management from the Mac

 Single-node k3s cluster on prodesk01 (192.168.1.70), managed from a MacBook with kubectl.

 ## 1. Install k3s (on prodesk01)

 ```bash
 curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--disable=helm-controller" sh -
 ```

 - `--disable=helm-controller` turns off the built-in Helm controller.
 - Traefik and the other default components stay enabled.

 Check that the service is running:

 ```bash
 sudo systemctl status k3s.service
 sudo k3s kubectl get nodes
 ```

 The node should be `Ready`.

 ## 2. Make the kubeconfig available (on prodesk01)

 k3s writes its config to `/etc/rancher/k3s/k3s.yaml`, readable by root only.
 Copy it to the home directory so scp can fetch it:

 ```bash
 sudo cp /etc/rancher/k3s/k3s.yaml ~/k3s.yaml
 sudo chown pim:pim ~/k3s.yaml
 ```

 Check the home directory with `pwd` (here: `/home/pim`).

 ## 3. Fetch the kubeconfig (on the Mac)

 This command runs on the Mac; the HP is the source, the current folder on the Mac the destination.

 ```bash
 cd ~/.kube
 cp config config.old          # backup of any existing config
 scp pim@192.168.1.70:~/k3s.yaml ./config
 ```

 Change the IP address in the config. The file contains `127.0.0.1`, which on the Mac
 points to the Mac itself:

 ```bash
 sed -i '' 's/127.0.0.1/192.168.1.70/' config
 ```

 (`-i ''` is macOS/BSD syntax. On Linux it is `sed -i`.)

 Check:

 ```bash
 grep server config            # should show https://192.168.1.70:6443
 ```

 ## 4. Test (on the Mac)

 ```bash
 kubectl get nodes
 kubectl get pods -A
 ```

 The config has context `default`. Pick a specific context with
 `kubectl --context <name> get nodes` (the value after `--context` is required).

 ## 5. Common problems

 | Symptom | Cause / fix |
 |---|---|
 | `connection refused` or hang | Port 6443 not reachable. Test: `nc -zv 192.168.1.70 6443`. Check the firewall on the HP (`sudo ufw allow 6443/tcp`). |
 | TLS error about the IP address | IP is not in the server certificate. Install with `--tls-san 192.168.1.70` in `INSTALL_K3S_EXEC`. |
 | `Unauthorized` / certificate error | Old config from a previous install. Redo steps 2 and 3. |
 | `flags cannot be placed before plugin name` | `--context` given without a value. |

 ## 6. Notes

 - The kubeconfig contains the client certificate and key: the credentials for the cluster.
   Don't share them in chats, issues or commits.
   - After reinstalling k3s all certificates are regenerated, so the kubeconfig on the Mac
     must be fetched again.
     - To remove k3s: `/usr/local/bin/k3s-uninstall.sh` (on prodesk01).
