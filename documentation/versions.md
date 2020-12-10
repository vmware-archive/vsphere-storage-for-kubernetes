---
title: Version Features
summary: "This documentation is not applicable to vSphere CSI Driver. Please visit https://vsphere-csi-driver.sigs.k8s.io/ for information about vSphere CSI Driver."
---

The following table summarizes key features introduced in vSphere Cloud Provider in each Kubernetes release

<table>
<thead>
<tr>
  <th>Kubernetes Release</th>
  <th>vSphere Cloud Provider feature</th>
</tr>
</thead>
<tbody>
<tr>
  <td>v1.11.0</td>
  <td>Added a mechanism in vSphere Cloud Provider to get credentials from Kubernetes secrets, rather than the plain text vsphere.conf file<br>SAML token authentication support</td>
</tr>
<tr>
  <td>v1.9.0 </td>
  <td>Multi vCenter Support</td>
</tr>
<tr>
  <td>v1.8.2</td>
  <td>Performance improvement for large scale deployment</td>
</tr>
<tr>
  <td>v1.8.0</td>
  <td>vSphere Cloud Provider refactoring for better debuggability, logging and code maintenance</td>
</tr>
<tr>
  <td>v1.7.0</td>
  <td>Integration with vSphere Storage Policy Based Management (SPBM) for dynamic volume provisioning.<br>Enhanced vSphere Cloud Provider debuggabilty via integration with metrices exposed for Kubernetes storage APIs.</td>
</tr>
<tr>
  <td>v1.6.5</td>
  <td>Integration with vSphere HA</td>
</tr>
<tr>
  <td>v1.6.3</td>
  <td>Dynamic volume provisioning using vSAN storage capabilities</td>
</tr>
</tbody>
</table>
