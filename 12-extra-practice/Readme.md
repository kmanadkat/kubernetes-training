# Extra Practice (Grade Submission API)

Grade submission API pod

<div style="padding: 40px; display: flex; justify-content: center; font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;">
    <div style="width: 580px; background-color: #c0504d; padding: 15px 20px 25px 20px; border: 1px solid #333; color: white;">
        <div style="display: flex; justify-content: space-between; align-items: center; font-weight: bold; font-size: 19px; margin-bottom: 20px;">
            <span>grade-submission-api</span>
            <span style="font-size: 14px; font-weight: normal; opacity: 0.9;">Label Groupings</span>
        </div>
        <div style="display: flex; gap: 15px; align-items: flex-start;">
            <div style="width: 320px; border: 1px solid #111;">
                <div style="background-color: #004d80; padding: 10px; text-align: center; font-weight: bold; font-size: 18px;">Container</div>
                <div style="background-color: #1a1a1a; padding: 18px; display: flex; justify-content: center;">
                    <div style="background-color: #666; padding: 10px; width: 100%; text-align: center; font-size: 15px; color: white;">Grade Submission API</div>
                </div>
            </div>
            <div style="display: flex; flex-direction: column; gap: 15px;">
                <div style="background-color: #00b050; padding: 4px 12px; font-weight: bold; font-size: 20px; width: fit-content; color: white;">3000</div>
                <div style="background-color: #333; padding: 10px; font-size: 14px; border: 1px solid #444; color: #ccc;">Resource Requirements</div>
            </div>
        </div>
    </div>
</div>
<div style="padding: 40px; display: flex; justify-content: center; font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;">
    <div style="width: 580px; background-color: #c0504d; padding: 15px 20px 25px 20px; border: 1px solid #333; color: white;">
        <div style="display: flex; justify-content: space-between; align-items: center; font-weight: bold; font-size: 19px; margin-bottom: 20px;">
            <span>grade-submission-api</span>
            <span style="font-size: 14px; font-weight: normal; opacity: 0.9;">Label Groupings</span>
        </div>
        <div style="display: flex; gap: 15px;">
            <!-- Left: two containers -->
            <div style="flex: 1; display: flex; flex-direction: column; gap: 12px;">
                <!-- Container 1 -->
                <div style="border: 1px solid #111;">
                    <div style="background-color: #004d80; padding: 10px; text-align: center; font-weight: bold; font-size: 18px;">Container</div>
                    <div style="background-color: #1a1a1a; padding: 18px; display: flex; justify-content: center;">
                        <div style="background-color: #666; padding: 10px; width: 100%; text-align: center; font-size: 15px; color: white;">Grade Submission API</div>
                    </div>
                </div>
                <!-- Container 2 -->
                <div style="border: 1px solid #111;">
                    <div style="background-color: #004d80; padding: 10px; text-align: center; font-weight: bold; font-size: 18px;">Container</div>
                    <div style="background-color: #1a1a1a; padding: 18px; display: flex; justify-content: center;">
                        <div style="background-color: #666; padding: 10px; width: 100%; text-align: center; font-size: 15px; color: white;">Grade Submission API Health Checker</div>
                    </div>
                </div>
            </div>
            <!-- Right: port + line + localhost, aligned to container 1 top and container 2 center -->
            <div style="width: 80px; display: flex; flex-direction: column; align-items: center;">
                <!-- 5001 badge -->
                <div style="background-color: #00b050; padding: 4px 12px; font-weight: bold; font-size: 20px; color: white; white-space: nowrap;">3000</div>
                <!-- Line that goes down -->
                <div style="width: 2px; background-color: white; height: 100%;  flex: 1;"></div>
                <!-- localhost sits at the bottom, aligned with mid of container 2 -->
                <div style="font-size: 20px; font-weight: bold; color: white; padding-bottom: 30px;">localhost</div>
            </div>
        </div>
    </div>
</div>


```yaml
apiVersion: v1
kind: Pod
metadata:
  name: grade-submission-api
  labels:
    app.kubernetes.io/name: grade-submission
    app.kubernetes.io/component: backend
    app.kubernetes.io/instance: grade-submission-api

spec:
  containers:
    - name: grade-submission-api
      image: rslim087/kubernetes-course-grade-submission-api:stateless
      resources:
        requests:
          memory: '128Mi'
          cpu: '128m'
        limits:
          memory: '128Mi'
      ports:
        - containerPort: 3000

    - name: grade-submission-api-health-checker
      image: rslim087/kubernetes-course-grade-submission-api-health-checker
      resources:
        requests:
          memory: '128Mi'
          cpu: '128m'
        limits:
          memory: '128Mi'

```

>  If image tag is not specified, it's going to pull default tag `latest`



<div style="display: flex; justify-content: center;">
<div style="padding: 10px; display: flex; flex-direction: column; justify-content: center; font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;">
  <p style="font-weight: bold; font-size: 19px;">
    Network Namespace
  </p>
    <div style="width: 580px; background-color: #c0504d; padding: 15px 20px 25px 20px; border: 1px solid #333; color: white;">
        <div style="display: flex; justify-content: space-between; align-items: center; font-weight: bold; font-size: 19px; margin-bottom: 20px;">
            <span>grade-submission-portal</span>
            <span style="font-size: 14px; font-weight: normal; opacity: 0.9;">Label Groupings</span>
        </div>
        <div style="display: flex; gap: 15px;">
            <!-- Left: two containers -->
            <div style="flex: 1; display: flex; flex-direction: column; gap: 12px;">
                <!-- Container 1 -->
                <div style="border: 1px solid #111;">
                    <div style="background-color: #004d80; padding: 10px; text-align: center; font-weight: bold; font-size: 18px;">Container</div>
                    <div style="background-color: #1a1a1a; padding: 18px; display: flex; justify-content: center;">
                        <div style="background-color: #666; padding: 10px; width: 100%; text-align: center; font-size: 15px; color: white;">Grade Submission Portal</div>
                    </div>
                </div>
                <!-- Container 2 -->
                <div style="border: 1px solid #111;">
                    <div style="background-color: #004d80; padding: 10px; text-align: center; font-weight: bold; font-size: 18px;">Container</div>
                    <div style="background-color: #1a1a1a; padding: 18px; display: flex; justify-content: center;">
                        <div style="background-color: #666; padding: 10px; width: 100%; text-align: center; font-size: 15px; color: white;">Grade Submission Portal Health Checker</div>
                    </div>
                </div>
            </div>
            <!-- Right: port + line + localhost, aligned to container 1 top and container 2 center -->
            <div style="width: 80px; display: flex; flex-direction: column; align-items: center;">
                <!-- 5001 badge -->
                <div style="background-color: #00b050; padding: 4px 12px; font-weight: bold; font-size: 20px; color: white; white-space: nowrap;">5001</div>
                <!-- Line that goes down -->
                <div style="width: 2px; background-color: white; height: 100%;  flex: 1;"></div>
                <!-- localhost sits at the bottom, aligned with mid of container 2 -->
                <div style="font-size: 20px; font-weight: bold; color: white; padding-bottom: 30px;">localhost</div>
            </div>
        </div>
    </div>
</div>
<div style="padding: 10px; display: flex; justify-content: center; flex-direction: column; font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;">
  <p style="font-weight: bold; font-size: 19px;">
    Network Namespace
  </p>
    <div style="width: 580px; background-color: #c0504d; padding: 15px 20px 25px 20px; border: 1px solid #333; color: white;">
        <div style="display: flex; justify-content: space-between; align-items: center; font-weight: bold; font-size: 19px; margin-bottom: 20px;">
            <span>grade-submission-api</span>
            <span style="font-size: 14px; font-weight: normal; opacity: 0.9;">Label Groupings</span>
        </div>
        <div style="display: flex; gap: 15px;">
            <!-- Left: two containers -->
            <div style="flex: 1; display: flex; flex-direction: column; gap: 12px;">
                <!-- Container 1 -->
                <div style="border: 1px solid #111;">
                    <div style="background-color: #004d80; padding: 10px; text-align: center; font-weight: bold; font-size: 18px;">Container</div>
                    <div style="background-color: #1a1a1a; padding: 18px; display: flex; justify-content: center;">
                        <div style="background-color: #666; padding: 10px; width: 100%; text-align: center; font-size: 15px; color: white;">Grade Submission API</div>
                    </div>
                </div>
                <!-- Container 2 -->
                <div style="border: 1px solid #111;">
                    <div style="background-color: #004d80; padding: 10px; text-align: center; font-weight: bold; font-size: 18px;">Container</div>
                    <div style="background-color: #1a1a1a; padding: 18px; display: flex; justify-content: center;">
                        <div style="background-color: #666; padding: 10px; width: 100%; text-align: center; font-size: 15px; color: white;">Grade Submission API Health Checker</div>
                    </div>
                </div>
            </div>
            <!-- Right: port + line + localhost, aligned to container 1 top and container 2 center -->
            <div style="width: 80px; display: flex; flex-direction: column; align-items: center;">
                <!-- 5001 badge -->
                <div style="background-color: #00b050; padding: 4px 12px; font-weight: bold; font-size: 20px; color: white; white-space: nowrap;">3000</div>
                <!-- Line that goes down -->
                <div style="width: 2px; background-color: white; height: 100%;  flex: 1;"></div>
                <!-- localhost sits at the bottom, aligned with mid of container 2 -->
                <div style="font-size: 20px; font-weight: bold; color: white; padding-bottom: 30px;">localhost</div>
            </div>
        </div>
    </div>
</div>
</div>

- When we have two separate pods, they're automatically separated at the network level because of the network namespace.
- **Network Namespace** is a logical construct in Kubernetes that essentially isolates Kubernetes resources at the network level.

We want the great submission portal to forward that data to the Great Submission API, which is going to manage the submitted great data in the back end.





