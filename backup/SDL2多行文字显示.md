思路如下：
1. 按行分割多行字符串
2. 通过总体Surface将所有行Surface合并
```C++
void Simg::renderText(const std::string &str, SDL_Rect rect, SDL_Color color)
{
    // 分割str 按\n分割
    auto lines = splitLines(str);
    //标准行间距
    const int lineSkip = TTF_FontLineSkip(font);
    // 计算总高度
    int totalHeight = 0;
    // 计算最大宽度
    int maxWidth = 0;
    // 所有行surface
    std::vector<SDL_Surface*> lineSurfaces;
    for (const auto &str : lines)
    {
        SDL_Surface *surface = TTF_RenderUTF8_Blended(font, str.c_str(), color);
        if (surface == nullptr) continue;
        lineSurfaces.push_back(surface);
        surface->w > maxWidth ? maxWidth = surface->w : maxWidth;
        totalHeight += lineSkip;
    }
    // 创建目标表面（带透明度）
    SDL_Surface* final = SDL_CreateRGBSurfaceWithFormat(
        0, maxWidth, totalHeight, 32, SDL_PIXELFORMAT_RGBA32);
    if (final == nullptr)
    {
        for (auto s : lineSurfaces) SDL_FreeSurface(s);
        return;
    }
    // 设置透明背景
    SDL_FillRect(final, NULL, SDL_MapRGBA(final->format, 0, 0, 0, 0));

    // 逐行绘制文本
    int yPos = 0;
    for (auto surf : lineSurfaces) {
        SDL_Rect dest = {0, yPos, surf->w, surf->h};
        SDL_BlitSurface(surf, NULL, final, &dest);
        SDL_FreeSurface(surf); // 释放临时表面
        yPos += lineSkip;      // 移动到下一行位置
    }
    lineSurfaces.clear();

    SDL_Rect bgDst = {rect.x, rect.y, maxWidth, totalHeight};
    // 文字背景
    SDL_Color tcolor;
    SDL_GetRenderDrawColor(renderer, &tcolor.r, &tcolor.g, &tcolor.b, &tcolor.a);
    SDL_SetRenderDrawColor(renderer, (Uint8)(rgba[0] * 255), (Uint8)(rgba[1] * 255), (Uint8)(rgba[2] * 255), (Uint8)(rgba[3] * 255));
    SDL_RenderFillRect(renderer, &bgDst);
    SDL_SetRenderDrawColor(renderer, tcolor.r, tcolor.g, tcolor.b, tcolor.a);

    // Texture
    SDL_Texture *text_texture = SDL_CreateTextureFromSurface(renderer, final);
    SDL_FreeSurface(final);
    SDL_RenderCopyEx(renderer, text_texture, NULL, &bgDst, 0, NULL, SDL_RendererFlip::SDL_FLIP_NONE);
    SDL_DestroyTexture(text_texture);
}
```