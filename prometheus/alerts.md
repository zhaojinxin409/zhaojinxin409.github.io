# Proemtheus Alert 状态转换

## Alert相关结构

Alert是由AlertRule在运行期间生成的，每个AlertRule都会生成若干个告警，AlertRule的结构如下：

```golang

// 为了更好阐述AlertRule中告警相关的逻辑，对结构做了一些精简

type AlertingRule struct {
	// The name of the alert.
	name string
	// The vector expression from which to generate alerts.
	vector parser.Expr
	// The duration for which a labelset needs to persist in the expression
	// output vector before an alert transitions from Pending to Firing state.
	holdDuration time.Duration
	// Extra labels to attach to the resulting alert sample vectors.
	labels labels.Labels
	// Non-identifying key/value pairs.
	annotations labels.Labels
	// External labels from the global config.
	externalLabels map[string]string
	// The external URL from the --web.external-url flag.
	// A map of alerts which are currently active (Pending or Firing), keyed by
	// the fingerprint of the labelset they correspond to.
	active map[uint64]*Alert
}
```

AlertRule会由Manager调用定期运行`Eval`和`sendAlerts`函数用于评估是否有异常和发送告警请求。所有的告警都存储在active中。结构为Alert。

```golang
// Alert is the user-level representation of a single instance of an alerting rule.
type Alert struct {
	State AlertState

	Labels      labels.Labels
	Annotations labels.Labels

	// The value at the last evaluation of the alerting expression.
	Value float64
	// The interval during which the condition of this alert held true.
	// ResolvedAt will be 0 to indicate a still active alert.
	ActiveAt   time.Time
	FiredAt    time.Time
	ResolvedAt time.Time
	LastSentAt time.Time
	ValidUntil time.Time
}
```
