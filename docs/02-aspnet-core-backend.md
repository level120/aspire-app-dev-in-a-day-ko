# 세션 02: ASP.NET Core 백엔드 API 앱 개발

이 세션에서는 [ASP.NET Core 백엔드 API 앱](https://learn.microsoft.com/ko-kr/aspnet/core/fundamentals/apis?WT.mc_id=dotnet-121695-juyoo) 개발을 해 보겠습니다.

<!-- 이 세션에서는 [GitHub Copilot](https://docs.github.com/ko/copilot/overview-of-github-copilot/about-github-copilot-business) 기능을 활용해 빠르게 [ASP.NET Core 백엔드 API 앱](https://learn.microsoft.com/ko-kr/aspnet/core/fundamentals/apis?WT.mc_id=dotnet-121695-juyoo) 개발을 해 보겠습니다. -->

> [GitHub Codespaces](https://docs.github.com/ko/codespaces/overview) 환경에서 작업하는 것을 기준으로 진행합니다. 로컬 개발 환경의 [Visual Studio Code](https://code.visualstudio.com/?WT.mc_id=dotnet-121695-juyoo)를 사용할 경우 대부분 비슷하지만 살짝 다를 수 있습니다.

![Architecture](./images/02-architecture.png)

## 02-1: Web API 프로젝트 생성하기

> 세이브 포인트에서 가져온 프로젝트를 사용하려면 아래 명령어를 차례로 실행시켜 프로젝트를 복원합니다.
>
>    ```bash
>    cd $CODESPACE_VSCODE_FOLDER
>    mkdir -p workshop && cp -a save-points/session-01/. workshop/
>    cd workshop
>    dotnet restore && dotnet build
>    ```

1. 아래 명령어를 차례로 실행시켜 ASP.NET Core Web API 앱 프로젝트를 생성합니다.

    ```bash
    dotnet new webapi -n AspireYouTubeSummariser.ApiApp
    dotnet sln add AspireYouTubeSummariser.ApiApp
    ```

1. 아래 명령어를 차례로 실행시켜 ASP.NET Core Web API 앱 프로젝트를 빌드하고 실행시킵니다.

    ```bash
    dotnet restore && dotnet build
    dotnet run --project AspireYouTubeSummariser.ApiApp
    ```

1. 브라우저에서 404 에러가 나는 것을 확인합니다.
1. `Program.cs` 파일을 열고 `app.UseHttpsRedirection();` 라인을 찾아 그 바로 아래에 아래 코드를 추가합니다.
<!-- 1. `Program.cs` 파일을 열고 `app.UseHttpsRedirection();` 라인을 찾아 그 바로 아래에서 `CTRL`+`I` 키 또는 `CMD`+`I` 키를 눌러 GitHub Copilot Chat 창을 활성화 시킵니다.
1. 아래 프롬프트를 GitHub Copilot Chat에 입력합니다.

    ```text
    Add a minimal API logic with the following conditions:

    - It should redirect from "/" to "/swagger", when the app is running in development environment,
    - It should exclude this endpoint from the OpenAPI documentation.
    ```

   그러면 아래와 비슷한 코드가 생성되었을 것입니다. 아래 코드를 참고해서 `Program.cs` 파일을 수정합니다. -->

    ```csharp
    if (app.Environment.IsDevelopment())
    {
        app.MapGet("/", () => Results.Redirect("/swagger")).ExcludeFromDescription();
    }
    ```

## 02-2: Summary 엔드포인트 생성하기

1. `Program.cs` 파일을 열고 `app.MapGet("/weatherforecast", ...)` 메서드와 `app.Run();` 메서드 사이의 라인을 찾아 거기서 아래 코드를 입력합니다.
<!-- 1. `Program.cs` 파일을 열고 `app.MapGet("/weatherforecast", ...)` 메서드와 `app.Run();` 메서드 사이의 라인을 찾아 거기서 아래 프롬프트를 GitHub Copilot Chat에 입력합니다.

    ```text
    Add a minimal API logic with the following conditions:

    - It should be of the endpoint, "/summarise".
    - It should process the HTTP POST request.
    - It should accept the "SummaryRequest" object as a request body.
    - It should return a string of "hello world" as a response.
    - It should be named "GetSummary".
    - It should be documented in the OpenAPI documentation.
    ```

   그러면 아래와 비슷한 코드가 생성되었을 것입니다. 아래 코드를 참고해서 `Program.cs` 파일을 수정합니다. -->

    ```csharp
    app.MapPost("/summarise", ([FromBody] SummaryRequest req) =>
    {
        var summary = "Hello World!";
        return summary;
    })
    .WithName("GetSummary")
    .WithOpenApi();
    ```

1. `Program.cs` 파일의 맨 아랫 부분에 `SummaryRequest` 레코드를 생성합니다.

    ```csharp
    record SummaryRequest(string? YouTubeLinkUrl, string VideoLanguageCode, string? SummaryLanguageCode);
    ```

## 02-3: YouTubeSummariserService 생성하기

1. `Program.cs` 파일의 맨 아랫 부분에 아래와 같이 `YouTubeSummariserService` 클래스를 생성합니다.
<!-- 1. `Program.cs` 파일의 맨 아랫 부분에 `YouTubeSummariserService` 클래스를 생성합니다. GitHub Copilot Chat 창을 활성화 시킨 후 아래 프롬프트를 입력합니다.

    ```text
    Add a class of YouTubeSummariserService with the following conditions:

    - It should have a method, "SummariseAsync" with a parameter of SummaryRequest.
    - It should return a string.
    ```

   그러면 아래와 비슷한 코드가 생성되었을 것입니다. 아래 코드를 참고해서 `YouTubeSummariserService` 클래스를 수정합니다. -->

    ```csharp
    class YouTubeSummariserService
    {
        public async Task<string> SummariseAsync(SummaryRequest req)
        {
            string summary = "This is a summary of the YouTube video.";

            return await Task.FromResult(summary).ConfigureAwait(false);
        }
    }
    ```

1. `Program.cs` 파일에서 `var builder = WebApplication.CreateBuilder(args);` 라인을 찾아 바로 밑에 아래 코드를 추가합니다.

    ```csharp
    builder.Services.AddScoped<YouTubeSummariserService>();
    ```

1. 앞서 정의했던 `app.MapPost("/summarise", ...)` 메서드의 람다식을 아래와 같이 수정합니다.

    ```csharp
    app.MapPost("/summarise", async ([FromBody] SummaryRequest req, YouTubeSummariserService service) =>
    {
        var summary = await service.SummariseAsync(req);
        return summary;
    })
    .WithName("GetSummary")
    .WithOpenApi();
    ```

## 02-4: YouTubeSummariserService에 의존성 주입하기

1. YouTube 자막을 추출하고, Azure OpenAI 서비스를 이용해 요약하기 위해 아래와 같이 터미널에서 NuGet 패키지를 추가합니다.

    ```bash
    cd $CODESPACE_VSCODE_FOLDER/workshop/AspireYouTubeSummariser.ApiApp
    dotnet add package Aliencube.YouTubeSubtitlesExtractor
    dotnet add package Azure.AI.OpenAI --prerelease
    ```

1. `appsettings.json` 파일을 열고 아래와 같이 Azure OpenAI 서비스의 `Endpoint`와 `ApiKey`, `DeploymentName` 값을 추가합니다. 이 값들은 이미 이메일을 통해 받았을 것입니다.

    ```json
    "OpenAI": {
      "Endpoint": "{{ Azure OpenAI Proxy Service Endpoint }}",
      "ApiKey": "{{ Azure OpenAI Proxy Service Access Code }}",
      "DeploymentName": "{{ Azure OpenAI Proxy Service Deployment Name }}"
    }
    ```

   > **중요**: `appsettings.json` 파일에 추가한 Azure OpenAI 서비스의 값들은 절대로 GitHub에 커밋하지 마세요. 대신 `appsettings.Development.json` 파일에 추가하세요. `.gitignore` 파일에 이미 `appsettings.Development.json` 파일에 대한 제외 옵션이 추가되어 있습니다.

1. `appsettings.json` 파일에 아래와 같이 프롬프트 설정을 추가합니다.

    ```json
    "Prompt": {
      "System": "You are the expert of summarising long contents. You are going to summarise the following YouTube video transcript in a given language code.",
      "MaxTokens": 3000,
      "Temperature": 0.7
    },
    ```

1. `Program.cs` 파일을 열고 `YouTubeSummariserService` 클래스에 생성자를 아래와 같이 추가합니다.

    ```csharp
    class YouTubeSummariserService(IYouTubeVideo youtube, OpenAIClient openai, IConfiguration config)
    {
        private readonly IYouTubeVideo _youtube = youtube ?? throw new ArgumentNullException(nameof(youtube));
        private readonly OpenAIClient _openai = openai ?? throw new ArgumentNullException(nameof(openai));
        private readonly IConfiguration _config = config ?? throw new ArgumentNullException(nameof(config));
    ```

   > 만약 네임스페이스 참조를 할 수 없다는 오류가 발생한다면 오류가 발생한 곳에 커서를 두고 `CTRL`+`.` 키 또는 `CMD`+`.` 키를 눌러 네임스페이스를 추가합니다.

1. `SummariseAsync` 메서드를 아래와 같이 수정합니다.

    ```csharp
    public async Task<string> SummariseAsync(SummaryRequest req)
    {
        Subtitle subtitle = await this._youtube.ExtractSubtitleAsync(req.YouTubeLinkUrl, req.VideoLanguageCode).ConfigureAwait(false);
        string caption = subtitle.Content.Select(p => p.Text).Aggregate((a, b) => $"{a}\n{b}");
    ```

   > 만약 네임스페이스 참조를 할 수 없다는 오류가 발생한다면 오류가 발생한 곳에 커서를 두고 `CTRL`+`.` 키 또는 `CMD`+`.` 키를 눌러 네임스페이스를 추가합니다.

1. 계속해서 `SummariseAsync` 메서드 안에서 자막 내용을 요약하는 로직을 아래와 같이 구현합니다.

    ```csharp
    public async Task<string> SummariseAsync(SummaryRequest req)
    {
        ...

        ChatCompletionsOptions options = new()
        {
            DeploymentName = this._config["OpenAI:DeploymentName"],
            MaxTokens = Convert.ToInt32(this._config["Prompt:MaxTokens"]),
            Temperature = Convert.ToSingle(this._config["Prompt:Temperature"]),
        };
        options.Messages.Add(new ChatRequestSystemMessage(this._config["Prompt:System"]));
        options.Messages.Add(new ChatRequestSystemMessage($"Here's the transcript. Summarise it in 5 bullet point items in the given language code of \"{req.SummaryLanguageCode}\"."));
        options.Messages.Add(new ChatRequestUserMessage(caption));
    }
    ```

   > 만약 네임스페이스 참조를 할 수 없다는 오류가 발생한다면 오류가 발생한 곳에 커서를 두고 `CTRL`+`.` 키 또는 `CMD`+`.` 키를 눌러 네임스페이스를 추가합니다.

   마지막으로 아래 내용을 `SummariseAsync` 메서드 마지막에 추가합니다.

    ```csharp
    public async Task<string> SummariseAsync(SummaryRequest req)
    {
        ...

        var response = await this._openai.GetChatCompletionsAsync(options).ConfigureAwait(false);
        string summary = response.Value.Choices[0].Message.Content;

        return summary;
    }
    ```

1. `Program.cs` 파일의 `var builder = WebApplication.CreateBuilder(args);` 라인을 찾아 바로 밑에 아래 코드를 추가합니다.

    ```csharp
    builder.Services.AddHttpClient<IYouTubeVideo, YouTubeVideo>();
    builder.Services.AddScoped<OpenAIClient>(sp =>
    {
        var config = sp.GetRequiredService<IConfiguration>();
        var endpoint = new Uri(config["OpenAI:Endpoint"]);
        var credential = new AzureKeyCredential(config["OpenAI:ApiKey"]);
        var client = new OpenAIClient(endpoint, credential);

        return client;
    });
    ```

   > 만약 네임스페이스 참조를 할 수 없다는 오류가 발생한다면 오류가 발생한 곳에 커서를 두고 `CTRL`+`.` 키 또는 `CMD`+`.` 키를 눌러 네임스페이스를 추가합니다.

## 02-5: ASP.NET Core 백엔드 API 앱 실행하기

1. Solution Explorer에서 `AspireYouTubeSummariser.ApiApp` 프로젝트를 선택하고 마우스 오른쪽 버튼을 눌러 디버깅 모드로 실행합니다.

1. Swagger UI 화면에서 아래와 같이 `youTubeLinkUrl`, `videoLanguageCode`, `summaryLanguageCode` 값을 입력한 후 결과를 확인합니다.

    ```json
    {
      "youTubeLinkUrl": "https://youtu.be/z1M-7Bms1Jg?si=7sLzsBY00-vpf8xL",
      "videoLanguageCode": "en",
      "summaryLanguageCode": "ko"
    }
    ```

    ![Backend API #1](./images/02-aspnet-core-backend-01.png)

   > YouTube 링크는 무엇이든 상관 없습니다. 여기서는 [https://youtu.be/z1M-7Bms1Jg](https://youtu.be/z1M-7Bms1Jg) 링크를 사용합니다.

1. 그러면 아래와 같이 요약 결과가 나오는 것을 확인할 수 있습니다.

    ![Backend API #2](./images/02-aspnet-core-backend-02.png)

## 02-6: Blazor 프론트엔드 앱과 ASP.NET Core 백엔드 API 앱 통합하기

1. `AspireYouTubeSummariser.WebApp` 프로젝트의 `ApiAppClient` 클래스를 열고 아래 메서드를 추가합니다.

    ```csharp
    public async Task<List<WeatherForecast>> WeatherForecastAsync()
    {
        using var response = await _http.GetAsync("weatherforecast").ConfigureAwait(false);

        var forecasts = await response.Content.ReadFromJsonAsync<List<WeatherForecast>>().ConfigureAwait(false);
        return forecasts ?? [];
    }
    ```

1. `ApiAppClient` 클래스 안에 아래와 같이 서브클래스를 추가합니다.

    ```csharp
    public class WeatherForecast
    {
        public DateOnly Date { get; set; }
        public int TemperatureC { get; set; }
        public string? Summary { get; set; }
        public int TemperatureF => 32 + (int)(TemperatureC / 0.5556);
    }
    ```

1. `IApiAppClient` 인터페이스도 아래와 같이 수정합니다.

    ```csharp
    public interface IApiAppClient
    {
        Task<List<ApiAppClient.WeatherForecast>> WeatherForecastAsync();
        Task<string> SummariseAsync(string youTubeLinkUrl, string videoLanguageCode, string summaryLanguageCode);
    }
    ```

1. `Components/Pages` 디렉토리 안의 `Weather.razor` 파일을 열고 그 안의 `@code { ... }` 블록을 아래와 같이 수정합니다.

    ```razor
    @page "/weather"
    @using AspireYouTubeSummariser.WebApp.Clients

    // 추가
    @inject IApiAppClient ApiApp

    ...

    @code {
        private List<ApiAppClient.WeatherForecast>? forecasts;

        protected override async Task OnInitializedAsync()
        {
            forecasts = await ApiApp.WeatherForecastAsync();
        }
    }
    ```

   > 만약 네임스페이스 참조를 할 수 없다는 오류가 발생한다면 오류가 발생한 곳에 커서를 두고 `CTRL`+`.` 키 또는 `CMD`+`.` 키를 눌러 네임스페이스를 추가합니다.

1. 터미널에서 아래 명령어를 실행시켜 ASP.NET Core 백엔드 API 앱을 실행시킵니다.

    ```bash
    cd $CODESPACE_VSCODE_FOLDER/workshop
    dotnet run --project AspireYouTubeSummariser.ApiApp
    ```

1. Solution Explorer에서 `AspireYouTubeSummariser.WebApp` 프로젝트를 선택하고 마우스 오른쪽 버튼을 눌러 디버깅 모드로 실행합니다.

1. 첫 화면에서 아래와 같이 YouTube 링크를 입력하고 `Summarise!` 버튼을 클릭합니다.

    ![YouTubeSummariserComponent #1](./images/02-aspnet-core-backend-03.png)

   > YouTube 링크는 무엇이든 상관 없습니다. 여기서는 [https://youtu.be/z1M-7Bms1Jg](https://youtu.be/z1M-7Bms1Jg) 링크를 사용합니다.

1. 그러면 아래와 같이 요약 메시지가 나오는 것을 확인할 수 있습니다.

    ![YouTubeSummariserComponent #2](./images/02-aspnet-core-backend-04.png)

1. `Weather` 페이지로 이동해서 아래와 같이 날씨 정보가 나오는 것을 확인할 수 있습니다.

    ![Weather Page](./images/02-aspnet-core-backend-05.png)

---

축하합니다! ASP.NET Core 백엔드 API 앱 개발이 끝났습니다. 이제 [Session 03: Aspire 통합](./03-aspire-integration.md)으로 넘어가세요.
