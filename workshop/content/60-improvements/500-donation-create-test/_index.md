+++
title = "4.5 Donation create test"
chapter = true
weight = 500
+++

# Donation create test

Your time to shine again! Please proceed with implementing a test for the `/donation/create` endpoint and DB layer.
It is very similar to the existing test for the `/donor/signup` endpoint.

Once you finish the end result should show two successful tests.

![](/images/code_screenshots/60_500_1.png)

***

{{%expand "Solution" %}}
Quite similar to the test we already have

```
@mock.patch.dict(os.environ, {"TABLE_NAME": f"{first_name}-savealife-{ENV}"})
def test_donation_create():
    from app import app

    json_payload = {
        "city": "Haarlem",
        "type": "A+",
        "datetime": "2022-04-06T13:00:00",
        "address": "2nd street",
    }

    with Client(app) as client:
        response = client.http.post(
            "/donation/create",
            headers={"Content-Type": "application/json"},
            body=json.dumps(json_payload),
        )

        assert response.status_code == 200
        assert response.json_body.get("success")
```
{{% /expand%}}