



#### client-go入门之3：解析 yaml 文件并创建 k8s 资源对象

我们使用 k8s 提供的命令行工具 kubeclt 可以很方便地使用`kubectl apply -f xx.yaml`从 yaml 文件创建 k8s 的资源对象，但使用 client-go 的时候，通常是使用 clientset 的 list 或 get 接口先获取集群中已有的对象，再通过修改对象的属性之后调用 update 接口去操作集群中的资源对象，若要通过 client-go 使用 yaml 文件创建新的资源对象，则不能使用 clientset 而要使用 dynamicClient，这个用起来就没有 clientset 那么简单了，以下代码演示了通过 dynamicClient 使用 yaml 文件创建 k8s 资源的方法：

```shell
package main

import (
	"bytes"
	"context"
	"fmt"
	"io/ioutil"
	"k8s.io/apimachinery/pkg/api/meta"
	metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
	"k8s.io/apimachinery/pkg/apis/meta/v1/unstructured"
	"k8s.io/apimachinery/pkg/runtime"
	"k8s.io/apimachinery/pkg/runtime/serializer/yaml"
	yamlutil "k8s.io/apimachinery/pkg/util/yaml"
	"k8s.io/client-go/dynamic"
	"k8s.io/client-go/kubernetes"
	"k8s.io/client-go/restmapper"
	"k8s.io/client-go/tools/clientcmd"
	"log"
	"os/user"
)

func main() {
	home := GetHomePath()
	nameSpace := "demo01"
	k8sConfig, err := clientcmd.BuildConfigFromFlags("", fmt.Sprintf("%s/.kube/config", home))  // 使用 kubectl 默认配置 ~/.kube/config
	if err != nil {
		fmt.Printf("%v",err)
		return
	}

	// 创建一个k8s客户端
	clientSet, err := kubernetes.NewForConfig(k8sConfig)
	if err != nil {
		fmt.Printf("%v",err)
		return
	}
	dd, err := dynamic.NewForConfig(k8sConfig)
	if err != nil {
		log.Fatal(err)
	}

	filebytes, err := ioutil.ReadFile("demo.yaml")
	if err != nil {
		fmt.Printf("%v\n", err)
	}

	decoder := yamlutil.NewYAMLOrJSONDecoder(bytes.NewReader(filebytes), 100)
	for {
		var rawObj runtime.RawExtension
		if err = decoder.Decode(&rawObj); err != nil {
			break
		}

		obj, gvk, err := yaml.NewDecodingSerializer(unstructured.UnstructuredJSONScheme).Decode(rawObj.Raw, nil, nil)
		unstructuredMap, err := runtime.DefaultUnstructuredConverter.ToUnstructured(obj)
		if err != nil {
			log.Fatal(err)
		}

		unstructuredObj := &unstructured.Unstructured{Object: unstructuredMap}

		gr, err := restmapper.GetAPIGroupResources(clientSet.Discovery())
		if err != nil {
			log.Fatal(err)
		}

		mapper := restmapper.NewDiscoveryRESTMapper(gr)
		mapping, err := mapper.RESTMapping(gvk.GroupKind(), gvk.Version)
		if err != nil {
			log.Fatal(err)
		}

		var dri dynamic.ResourceInterface
		if mapping.Scope.Name() == meta.RESTScopeNameNamespace {
			if unstructuredObj.GetNamespace() == "" {
				unstructuredObj.SetNamespace(nameSpace)
			}
			dri = dd.Resource(mapping.Resource).Namespace(unstructuredObj.GetNamespace())
		} else {
			dri = dd.Resource(mapping.Resource)
		}

		obj2, err := dri.Create(context.Background(), unstructuredObj, metav1.CreateOptions{})
		if  err != nil {
			log.Fatal(err)
		}

		fmt.Printf("%s/%s created", obj2.GetKind(), obj2.GetName())
	}
}

func GetHomePath() string {
	u , err := user.Current()
	if err == nil {
		return u.HomeDir
	}
	return ""
}
```





k8s golang客户端client-go入门之4：解析yaml文件并创建k8s资源对象



