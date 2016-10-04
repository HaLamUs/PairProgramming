# PairProgramming
```Swift
struct URLSessionRequestSender: RequestSender {
    func send<T : Request>(_ r: T, handler: @escaping (Result) -> ()) {
        let url = URL(string: host.appending(r.path))!
        var request = URLRequest(url: url)
        request.httpMethod = r.method.rawValue
        if r.parameter.count > 0 {
            let jsonData = try? JSONSerialization.data(withJSONObject: r.parameter, options: [])
            request.setValue("application/json", forHTTPHeaderField: "Content-Type")
            request.httpBody = jsonData
        }
        let task = URLSession.shared.dataTask(with: request){
            data, res, error in
             guard error == nil else {
                print("Co loi roi \(error)")
                handler(.failure(error))
                return
            }
            let httpRes = res as! HTTPURLResponse
            guard httpRes.statusCode == 200 else {
                print("Loi \(httpRes.statusCode)")
                handler(.success(nil))
                return
            }
            if let data = data, let json = try? JSONSerialization.jsonObject(with: data, options: []){
                T.Response.parse(data: json as AnyObject)
                    handler(.success(json as AnyObject))
            } else {
                handler(.success(nil))
            }
        }
        task.resume()
    }
}
```
