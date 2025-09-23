# ❗The type or namaspace name 'Plastic' does not exist

{% hint style="danger" %}
**Error Details:**

The type or namaspace name 'Plastic' does not exist in the namespace 'Unity' (are you missing an assembly reference?)
{% endhint %}

{% hint style="success" %}
S**olution:**

Find the following line in files:

`using Unity.Plastic.Newtonsoft.Json;`

and replace it with the following:

`using Newtonsoft.Json;`
{% endhint %}
