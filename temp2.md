[{"op": "add", "path": "/metadata/annotations/aaaa", "value": "aaaaa"}]





kubectl patch deployment reviews --type='json' -p='[{"op": "add", "path": "/metadata/annotations/aaaa", "value": "aaaaa"}]'

kubectl patch deployment reviews --type='json' -p='[{"op": "replace", "path": "/spec/template/spec/containers/-", "value": [{"name": "aaa", "image":"adad"}]}]' -n nocalhost-test

[{"op":"replace","path":"/spec/suspend","value": "true"}]





/Users/xinxinhuang/.nh/bin/nhctl dev start bookinfo --deployment sample --local-sync /Users/xinxinhuang/WorkSpaces/other/bookinfo/reviews --container nginx --controller-type cloneset.v1alpha1.apps.kruise.io  --kubeconfig /Users/xinxinhuang/.nh/intellij-plugin/kubeConfigs/b52fee34-fbb7-4edb-a933-c4991f9cd07b_config --namespace nocalhost-test



nhctl dev start bookinfo --deployment sample --container nginx --controller-type cloneset.v1alpha1.apps.kruise.io  --kubeconfig /Users/xinxinhuang/.nh/intellij-plugin/kubeConfigs/b52fee34-fbb7-4edb-a933-c4991f9cd07b_config --namespace nocalhost-test

