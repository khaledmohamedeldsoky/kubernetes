# Volumes 

▶️ This data come from Conatainer ➡️ Pod ➡️ Node(only this node 💻 )

```yaml
apiVersion: v1
kind: Pod
metadata:
    name: random-number-generator
spec:
	containers:
	- image: alpine
		name: alpine
		command: ["/bin/sh","-c"]
		args: ["shuf -i 0-100 -n 1 >> /opt/number.out;"]
		volumeMounts:
		- mountPath: /opt # From the Container 
			name: data-volume  # To the Pod

	volumes:
	- name: data-volume # From the Pod
		hostPath:
			path: /data # To the Node
			type: Directory
```