



## hpcagent

入口程序`vendor/hpcagent/svc/main.go`

提交作业：`SlurmAgent.SubmitJob`



调用C接口向Slurm提交Job作业：

```c
/*
 * slurm_submit_batch_job - issue RPC to submit a job for later execution
 * NOTE: free the response using slurm_free_submit_response_response_msg
 * IN job_desc_msg - description of batch job request
 * OUT slurm_alloc_msg - response to request
 * RET SLURM_SUCCESS on success, otherwise return SLURM_ERROR with errno set
 */
extern int slurm_submit_batch_job(job_desc_msg_t *job_desc_msg,
				  submit_response_msg_t **slurm_alloc_msg);
```

C结构体job_desc_msg描述了向slurm提交的批处理任务信息：

```c
typedef struct job_descriptor {	/* For submit, allocate, and update requests */
	char *account;		/* charge to specified account */
	char *acctg_freq;	/* accounting polling intervals (seconds) */
	char *admin_comment;	/* administrator's arbitrary comment (update only) */
	char *alloc_node;	/* node making resource allocation request
				 * NOTE: Normally set by slurm_submit* or
				 * slurm_allocate* function */
	uint16_t alloc_resp_port; /* port to send allocation confirmation to */
	uint32_t alloc_sid;	/* local sid making resource allocation request
				 * NOTE: Normally set by slurm_submit* or
				 * slurm_allocate* function
				 * NOTE: Also used for update flags, see
				 * ALLOC_SID_* flags */
	uint32_t argc;		/* number of arguments to the script */
	char **argv;		/* arguments to the script */
	char *array_inx;	/* job array index values */
	void *array_bitmap;	/* NOTE: Set by slurmctld */
	char *batch_features;	/* features required for batch script's node */
	time_t begin_time;	/* delay initiation until this time */
	uint32_t bitflags;      /* bitflags */
	char *burst_buffer;	/* burst buffer specifications */
	char *clusters;		/* cluster names used for multi-cluster jobs */
	char *cluster_features;	/* required cluster feature specification,
				 * default NONE */
	char *comment;		/* arbitrary comment */
	uint16_t contiguous;	/* 1 if job requires contiguous nodes,
				 * 0 otherwise,default=0 */
	uint16_t core_spec;	/* specialized core/thread count,
				 * see CORE_SPEC_THREAD */
	char *cpu_bind;		/* binding map for map/mask_cpu - This
				 * currently does not matter to the
				 * job allocation, setting this does
				 * not do anything for steps. */
	uint16_t cpu_bind_type;	/* see cpu_bind_type_t - This
				 * currently does not matter to the
				 * job allocation, setting this does
				 * not do anything for steps. */
	uint32_t cpu_freq_min;  /* Minimum cpu frequency  */
	uint32_t cpu_freq_max;  /* Maximum cpu frequency  */
	uint32_t cpu_freq_gov;  /* cpu frequency governor */
	char *cpus_per_tres;	/* semicolon delimited list of TRES=# values */
	time_t deadline;	/* deadline */
	uint32_t delay_boot;	/* delay boot for desired node state */
	char *dependency;	/* synchronize job execution with other jobs */
	time_t end_time;	/* time by which job must complete, used for
				 * job update only now, possible deadline
				 * scheduling in the future */
	char **environment;	/* environment variables to set for job,
				 *  name=value pairs, one per line */
	uint32_t env_size;	/* element count in environment */
	char *extra;		/* unused */
	char *exc_nodes;	/* comma separated list of nodes excluded
				 * from job's allocation, default NONE */
	char *features;		/* required feature specification,
				 * default NONE */
	uint64_t fed_siblings_active; /* Bitmap of active fed sibling ids */
	uint64_t fed_siblings_viable; /* Bitmap of viable fed sibling ids */
	uint32_t group_id;	/* group to assume, if run as root. */
	uint32_t het_job_offset; /* HetJob component offset */
	uint16_t immediate;	/* 1 if allocate to run or fail immediately,
				 * 0 if to be queued awaiting resources */
	uint32_t job_id;	/* job ID, default set by Slurm */
	char * job_id_str;      /* string representation of the jobid */
	uint16_t kill_on_node_fail; /* 1 if node failure to kill job,
				     * 0 otherwise,default=1 */
	char *licenses;		/* licenses required by the job */
	uint16_t mail_type;	/* see MAIL_JOB_ definitions above */
	char *mail_user;	/* user to receive notification */
	char *mcs_label;	/* mcs_label if mcs plugin in use */
	char *mem_bind;		/* binding map for map/mask_cpu */
	uint16_t mem_bind_type;	/* see mem_bind_type_t */
	char *mem_per_tres;	/* semicolon delimited list of TRES=# values */
	char *name;		/* name of the job, default "" */
	char *network;		/* network use spec */
	uint32_t nice;		/* requested priority change,
				 * NICE_OFFSET == no change */
	uint32_t num_tasks;	/* number of tasks to be started,
				 * for batch only */
	uint8_t open_mode;	/* out/err open mode truncate or append,
				 * see OPEN_MODE_* */
	char *origin_cluster;	/* cluster name that initiated the job. */
	uint16_t other_port;	/* port to send various notification msg to */
	uint8_t overcommit;	/* over subscribe resources, for batch only */
	char *partition;	/* name of requested partition,
				 * default in Slurm config */
	uint16_t plane_size;	/* plane size when task_dist =
				   SLURM_DIST_PLANE */
	uint8_t power_flags;	/* power management flags,
				 * see SLURM_POWER_FLAGS_ */
	uint32_t priority;	/* relative priority of the job,
				 * explicitly set only for user root,
				 * 0 == held (don't initiate) */
	uint32_t profile;	/* Level of acct_gather_profile {all | none} */
	char *qos;		/* Quality of Service */
	uint16_t reboot;	/* force node reboot before startup */
	char *resp_host;	/* NOTE: Set by slurmctld */
	uint16_t restart_cnt;	/* count of job restarts */
	char *req_nodes;	/* comma separated list of required nodes
				 * default NONE */
	uint16_t requeue;	/* enable or disable job requeue option */
	char *reservation;	/* name of reservation to use */
	char *script;		/* the actual job script, default NONE */
	void *script_buf;	/* job script as mmap buf */
	uint16_t shared;	/* 2 if the job can only share nodes with other
				 *   jobs owned by that user,
				 * 1 if job can share nodes with other jobs,
				 * 0 if job needs exclusive access to the node,
				 * or NO_VAL to accept the system default.
				 * SHARED_FORCE to eliminate user control. */
	uint32_t site_factor;	/* factor to consider in priority */
	char **spank_job_env;	/* environment variables for job prolog/epilog
				 * scripts as set by SPANK plugins */
	uint32_t spank_job_env_size; /* element count in spank_env */
	uint32_t task_dist;	/* see enum task_dist_state */
	uint32_t time_limit;	/* maximum run time in minutes, default is
				 * partition limit */
	uint32_t time_min;	/* minimum run time in minutes, default is
				 * time_limit */
	char *tres_bind;	/* Task to TRES binding directives */
	char *tres_freq;	/* TRES frequency directives */
	char *tres_per_job;	/* semicolon delimited list of TRES=# values */
	char *tres_per_node;	/* semicolon delimited list of TRES=# values */
	char *tres_per_socket;	/* semicolon delimited list of TRES=# values */
	char *tres_per_task;	/* semicolon delimited list of TRES=# values */
	uint32_t user_id;	/* set only if different from current UID,
				 * can only be explicitly set by user root */
	uint16_t wait_all_nodes;/* 0 to start job immediately after allocation
				 * 1 to start job after all nodes booted
				 * or NO_VAL to use system default */
	uint16_t warn_flags;	/* flags  related to job signals
				 * (eg. KILL_JOB_BATCH) */
	uint16_t warn_signal;	/* signal to send when approaching end time */
	uint16_t warn_time;	/* time before end to send signal (seconds) */
	char *work_dir;		/* pathname of working directory */

	/* job constraints: */
	uint16_t cpus_per_task;	/* number of processors required for
				 * each task */
	uint32_t min_cpus;	/* minimum number of processors required,
				 * default=0 */
	uint32_t max_cpus;	/* maximum number of processors required,
				 * default=0 */
	uint32_t min_nodes;	/* minimum number of nodes required by job,
				 * default=0 */
	uint32_t max_nodes;	/* maximum number of nodes usable by job,
				 * default=0 */
	uint16_t boards_per_node; /* boards per node required by job  */
	uint16_t sockets_per_board;/* sockets per board required by job */
	uint16_t sockets_per_node;/* sockets per node required by job */
	uint16_t cores_per_socket;/* cores per socket required by job */
	uint16_t threads_per_core;/* threads per core required by job */
	uint16_t ntasks_per_node;/* number of tasks to invoke on each node */
	uint16_t ntasks_per_socket;/* number of tasks to invoke on
				    * each socket */
	uint16_t ntasks_per_core;/* number of tasks to invoke on each core */
	uint16_t ntasks_per_board;/* number of tasks to invoke on each board */
	uint16_t pn_min_cpus;    /* minimum # CPUs per node, default=0 */
	uint64_t pn_min_memory;  /* minimum real memory per node OR
				  * real memory per CPU | MEM_PER_CPU,
				  * default=0 (no limit) */
	uint32_t pn_min_tmp_disk;/* minimum tmp disk per node,
				  * default=0 */

	uint32_t req_switch;    /* Minimum number of switches */
	dynamic_plugin_data_t *select_jobinfo; /* opaque data type,
					   * Slurm internal use only */
	char *std_err;		/* pathname of stderr */
	char *std_in;		/* pathname of stdin */
	char *std_out;		/* pathname of stdout */
	uint64_t *tres_req_cnt; /* used internally in the slurmctld,
				   DON'T PACK */
	uint32_t wait4switch;   /* Maximum time to wait for minimum switches */
	char *wckey;            /* wckey for job */
	uint16_t x11;		/* --x11 flags */
	char *x11_magic_cookie;	/* automatically stolen from submit node */
	char *x11_target;	/* target hostname, or unix socket if port == 0 */
	uint16_t x11_target_port; /* target tcp port, 6000 + the display number */
} job_desc_msg_t;
```

对应的go结构体：SLURMJobDescMsg：

```go
type SLURMJobDescMsg struct {
	//  作业基本信息
	Name        string   `json:"job-name,omitempty"`
	WorkDir     string   `json:"chdir,omitempty"`
	Script      string   `json:"script"`
	Argv        []string `json:"argv,omitempty"`
	Environment []string `json:"export,omitempty"`
	Comment     string   `json:"comment,omitempty"`
	Deadline    int64    `json:"deadline,omitempty"`
	TimeLimit   uint32   `json:"time,omitempty"`
	StdIn       string   `json:"input,omitempty"`
	StdErr      string   `json:"error,omitempty"`
	StdOut      string   `json:"output,omitempty"`
	// 用户信息
	UserId  uint32 `json:"uid"`
	GroupId uint32 `json:"gid"`
	Account string `json:"account,omitempty"`
	// 资源信息
	Partition string `json:"partition,omitempty"`
	ExcNodes  string `json:"exclude,omitempty"`
	Nodelist  string `json:"nodelist,omitempty"`
	MinNodes  uint32 `json:"nodes,omitempty"`
	MaxNodes  uint32 `json:"maxnodes,omitempty"`

	NTasksPerNode uint16 `json:"ntasks-per-node,omitempty"`
	CPUsPerTask   uint16 `json:"cpus-per-task,omitempty"`
	NumTasks      uint32 `json:"ntasks,omitempty"`
	PnMinCPUs     uint16 `json:"mincpus,omitempty"`
	PnMinMemory   uint64 `json:"mem,omitempty"`
	// 行为控制
	Requeue        uint16 `json:"requeue,omitempty"`
	Reboot         uint16 `json:"reboot,omitempty"`
	KillOnNodeFail uint16 `json:"no-kill,omitempty"`
	Shared         uint16 `json:"share,omitempty"`
	WaitAllNodes   uint16 `json:"wait-all-nodes,omitempty"`
	OpenMode       uint8  `json:"open-mode,omitempty"`
	Nice           int32  `json:"nice,omitempty"`
	// 以下是 sbatch 输入参数中没有，而 job_desc_msg_t 结构体中存在的
	MinCPUs uint32
	MaxCPUs uint32 `json:"maxcpus,omitempty"` // sbatch代码似乎没有涉及这个值
	EnvSize uint32 `json:"envsize,omitempty"`
	Argc    uint32 `json:"argc,omitempty"`
}
```



SubmitJobRequest:

```json
UserID:1870 GroupID:1884 User:"nscc-gz_jiangli" Group:"nscc-gz" Partition:"power" Name:"simpleHpcJ-2162247" WorkDir:"/GPUFS/nscc-gz_jiangli/.bihu/wfda97f4b-5923-415f-9951-a7dca9b54c9f" Script:"#!/bin/bash\n\n\n\nenv\nhostname\nwhoami\nsleep 3600\ndate\n\n\n\n" Environment:"HOME=/WORK/nscc-gz_jiangli" Nodes:1 Options:<key:"slurm" value:<Comment:"{\"sid\":\"wfda97f4b-5923-415f-9951-a7dca9b54c9f\",\"app\":\"simpleHpcJobHour\"}" > >
```



`https://starlight-pre.nscc-gz.cn/api/job/submit`请求参数：

```json
{"app":"jupyter_HPC","params":{"haha":""},"runtime_params":{"undefined":"","endpoints":[{"control":1,"name":"jupyter-default","protocol":1,"source_port":8888,"target_port":8888,"type":1}],"env":{},"jobname":"jupyter_HP-9153134","partition":"bigdata","node":1,"cluster":"tianhe2h"}}
```

参数会被封装到：

```go
type JobSummitWrap struct {
	App          string `json:"app"`                     // App 名称
	WorkflowUUID string `json:"workflow_uuid,omitempty"` //
	//Params map[string]interface{} `json:"params,omitempty"`// 参数设置
	Params json.RawMessage `json:"params,omitempty"` // 参数设置
	//RuntimeParams map[string]interface{} `json:"runtime_params,omitempty"`// 运行时参数设置
	RuntimeParams json.RawMessage `json:"runtime_params,omitempty"` // 运行时参数设置
}
```

请求参数+用户信息会被封装到：

```go
type WorkFlow struct {
	*model.WorkFlowBasic
	app        *model.App
	tool       *cwl.Tool
	values     cwl.Values
	Chriden    []string // 子工作流
	TaskIndexs []string // 子任务
}

type WorkFlowBasic struct {
	Uuid        string `json:"uuid" gorm:"column:uuid;type:varchar(64);not null;primary_key"`   // 唯一标示
	UserLDAPUID int    `json:"user_ldap_uid" gorm:"column:user_ldap_uid;type:int(11);not null"` // 系统用户LDAP UID
	//UserName      string  `json:"user_name" gorm:"-"`               // 系统用户名，自动补充
	GroupName string         `json:"group_name" gorm:"-"`                                           // 租户名称，自动补充
	GroupId   int            `json:"group_id" gorm:"-"`                                             // 系统用户LDAP GID
	UserHome  string         `json:"user_home" gorm:"-"`                                            // 系统用户LDAP HomeDir
	Name      string         `json:"name" gorm:"column:name;type:varchar(64);not null"`             // 用户自定义名称
	CreatedBy string         `json:"created_by" gorm:"column:created_by;type:varchar(64);not null"` // 创建用户名
	CreatedAt Timestamp      `json:"created_at" gorm:"column:created_at;type:bigint(20);not null"`  // 创建时间
	Status    WorkFlowStatus `json:"status" gorm:"column:status;type:varchar(32);not null"`         // 状态
	Class     WorkFlowType   `json:"class" gorm:"column:class;type:varchar(32);not null"`           // 工作流类型
	//Raw *string // 工作流描述
	AppName *string `json:"app_name" gorm:"column:app_name;type:varchar(64);not null"` // 对应的 App 对象ID
	//ParamsRaw *string // 工作流参数
	//Params Params `json:"params" gorm:"column:params;type:text;default null"`  // 工作流参数,用于提交
	Params json.RawMessage `json:"params" gorm:"column:params;type:text;default null"` // 工作流参数,用于提交

	//RuntimeParams Params `json:"runtime_params" gorm:"column:runtime_params;type:text;default null"`// 工作流参数,用于运行时
	RuntimeParams json.RawMessage `json:"runtime_params" gorm:"column:runtime_params;type:text;default null"` // 工作流参数,用于运行时
	// TODO add RootWorkflow *string
	Parent      *string   `json:"parent" gorm:"column:parent;type:varchar(64);default null"`    // 父工作流，用来描述嵌套
	RetryConfig *Retry    `json:"retry" gorm:"column:retry;type:text;default null"`             // 重试设置
	AlertConfig *Alert    `json:"alert" gorm:"column:alert;type:text;default null"`             // 提醒设置
	UpdatedAt   Timestamp `json:"updated_at" gorm:"column:updated_at;type:bigint(20);not null"` // 更新时间
	//Agent string
	Version int64 `json:"version" gorm:"column:version;type:bigint(20);not null"` // 乐观锁

}
```

调用`job.AddWorkFlow(w *WorkFlow)`会把这个任务信息添加到工作流引擎中：

```go
func AddWorkFlow(w *WorkFlow) error {
	return workflowEngine.Add(w)
}
```

这个任务信息会被存放到一个map中：

```go
workflowMap[w.Uuid] = w
```

然后开启一个go routine调用`job.StartWorkflow(workflow)`运行该任务。

submitTool(w, w.tool, w.values)，在这个方法里，job会被包装成一个task：

```go
type Task struct {
	Job *model.Job
	app *model.App
	*model.TaskBasic
	RuntimeParams model.Params
	Values        *cwl.Values
	Outputs       cwl.Values
	Tool          *cwl.Tool
	//ToolRaw json.RawMessage
	ToolRaw string
	//ValueRaw json.RawMessage
	Prepares []string
	Nexts    []string
}
```

调用taskEngine.StartTask(task)，选择一个agent，

创建一个StagedTask：

```go
type StagedTask struct {
	*Stage
	*StageTaskPlan
	Inputs, Outputs       []File
	Volumes               []string
	Stdin, Stdout, Stderr string
	//State TaskState
}

type Stage struct {
	Dir      string // local real dir
	RDir     string // for local run
	Mode     os.FileMode
	LeaveDir bool
}

type StageTaskPlan struct {
	ID             string
	ContainerImage string
	Command        []string
	Env            map[string]string
	Workdir        string
	//Volumes []string
	Volumes []model.Volume
	Inputs,
	// All output paths must be contained in a volume.
	Outputs []File
	Stdin, Stdout, Stderr string
}

type Volume struct {
	Name string `yaml:"name" json:"name"`
	//HostPath  string `yaml:"host_path",json:"host_path"`
	HostPath  string `json:"host_path" yaml:"host_path"`
	ReadOnly  bool   `yaml:"read_only" json:"read_only"`
	MountPath string `yaml:"mount_path" json:"mount_path"`
}
```

K8sJob.Start(ctx context.Context, task *core.StagedTask, stdio *core.Stdio) (jobindex string, err error) 会根据集群信息选择一个`kubernetes.Clientset`，生成yaml，yaml文件信息类似下面：

```yaml
metadata:
  creationTimestamp: null
  labels:
    app: deeplearningtest
    jobname: deeplearni-9165855
    name: wbaf1cd1b-94c0-44a3-bb88-3b9906dd6cf4
    partition: power
  name: wbaf1cd1b-94c0-44a3-bb88-3b9906dd6cf4
spec:
  replicas: 1
  selector:
    matchLabels:
      name: wbaf1cd1b-94c0-44a3-bb88-3b9906dd6cf4
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: deeplearningtest
        jobname: deeplearni-9165855
        name: wbaf1cd1b-94c0-44a3-bb88-3b9906dd6cf4
        partition: power
    spec:
      containers:
      - command:
        - sleep
        - "3600"
        env:
        - name: CWD
          value: /GPUFS/nscc-gz_yfb_xxhuang/.bihu/wbaf1cd1b-94c0-44a3-bb88-3b9906dd6cf4
        - name: TMPDIR
          value: /tmp
        - name: SL_SECURE
          value: kK1OTvKoEjiC+FvwV544yGDW8bdz18UlNdBlGjdZn6U=
        - name: SL_PASSWD
          value: nscc-gz_yfb_xxhuang:x:10271:9865:nscc-gz_yfb:/BIGDATA1/nscc-gz_yfb_xxhuang:/bin/bash
        - name: USER
          value: nscc-gz_yfb_xxhuang
        - name: HOSTNAME
          value: deeplearningtest-wbaf1cd1b
        image: hub.starlight-dev.nscc-gz.cn/nscc-gz_jiangli_public/deeplearning-1586402268:centos
        imagePullPolicy: Always
        name: wbaf1cd1b-94c0-44a3-bb88-3b9906dd6cf4
        resources:
          limits:
            cpu: "1"
            memory: 4Gi
            nvidia.com/gpu: "0"
          requests:
            cpu: "1"
            memory: 4Gi
            nvidia.com/gpu: "0"
        securityContext:
          capabilities:
            add:
            - SYS_ADMIN
        volumeMounts:
        - mountPath: /app
          name: fsapp
          readOnly: true
        - mountPath: /GPUFS/nscc-gz_yfb_xxhuang
          name: fshome
        - mountPath: /GPUFS/nscc-gz_yfb_xxhuang/.bihu/wbaf1cd1b-94c0-44a3-bb88-3b9906dd6cf4
          name: workdir
        - mountPath: /dev/shm
          name: shm
        workingDir: /GPUFS/nscc-gz_yfb_xxhuang/.bihu/wbaf1cd1b-94c0-44a3-bb88-3b9906dd6cf4
      imagePullSecrets:
      - name: harbor-secret
      nodeSelector:
        partition: power
      volumes:
      - hostPath:
          path: /GPUFS/APP
        name: fsapp
      - hostPath:
          path: /GPUFS/nscc-gz_yfb_xxhuang
        name: fshome
      - hostPath:
          path: /GPUFS/nscc-gz_yfb_xxhuang/.bihu/wbaf1cd1b-94c0-44a3-bb88-3b9906dd6cf4
        name: workdir
      - emptyDir:
          medium: Memory
          sizeLimit: "2147483648"
        name: shm
```

deploy, err = client.AppsV1().Deployments(d.Namespace).Create(targetDeploy),



## TODO

- [ ] 如何得到yml信息
- [x] batch
- [ ] 查询k8s job信息






