动态类型信息(Dynamic Type Info)，下称DTI。

# 基本使用
- 通过定义原始基类DTI，存储类信息，在实际功能类中定义静态变量，实现类树的向上强制类型转换
- 使用`dynamic_cast`实现向下强制类型转换，注意要启用编译器的`GR`开关，并且放在`try/catch`中。使用`dynamic_cast`存在限制，此处不实现
```C++
#include <iostream>
#include <string>
/////////////////////////////////////
//H文件

/// @brief 动态类型信息
class DTI
{
public:
    DTI() = default;
    DTI(const std::string &name, DTI *parent)
        : _name(name), _parent(parent) {};
    ~DTI() = default;

    /// @brief 判断当前类是否是目标类型的子类
    /// @param pType 目标类型
    /// @return 是否是目标类型的子类
    bool IsA(DTI* pType);

// getters and setters
public:
    inline const std::string &getName() const { return this->_name; }
    inline const DTI *getParent() const { return this->_parent; }

    inline void setName(const std::string &name) { this->_name = name; }
    inline void setParent(DTI *parent) { this->_parent = parent; }

private:
    /// @brief 名称
    std::string _name;
    /// @brief 父类信息
    DTI *_parent{nullptr};
};

/// @brief 定义类型信息
#define EXPORT_TYPE \
public:             \
    static DTI Type;

/////////////////////////////////////
//CPP文件

bool DTI::IsA(DTI *pType)
{
    const DTI* current = this;
    while (current != nullptr)
    {
        if(current == pType)
            return true;
        else
            current = current->getParent();
    }
    return false;
}

////////////////////////////////////
//USE使用时

class CRootClass
{
    EXPORT_TYPE
public:
    CRootClass() = default;
    ~CRootClass() = default;
    /// @brief 如果存在频繁使用强制类型转换的时候，可以在基类实现一个安全的类型转换接口
    /// @brief 安全的类型转换
    /// @param pCastToType 目标类型
    /// @return 转换后的指针，如果转换失败，返回nullptr
    void* SafeCast(DTI* pCastToType);

    /// @brief 名称 用于SafeCast示例
    std::string name{"CRootClass"};
};

DTI CRootClass::Type{"CRootClass", nullptr};
void *CRootClass::SafeCast(DTI *pCastToType)
{
    if (this->Type.IsA(pCastToType))
        return this;
    return nullptr;
}

class CChildClass : public CRootClass
{
    EXPORT_TYPE
public:
    CChildClass() = default;
    ~CChildClass() = default;

    /// @brief 名称 用于SafeCast示例
    std::string name{"CChildClass"};
};

DTI CChildClass::Type{"CChildClass", &CRootClass::Type};

int main()
{
    // 示例1 子类直接查询父类名称
    std::cout << "Type name: " << CChildClass::Type.getName() << std::endl;
    std::cout << " Parent type name: "
              << CChildClass::Type.getParent()->getName() << std::endl;

    // 示例2 子类查询是否属于某个父类
    CChildClass child;
    if(child.Type.IsA(&CRootClass::Type))
    {
        std::cout << "child is a CRootClass" << std::endl;
    }
    // 示例3 子类强制转换为父类 若转换失败，返回nullptr
    auto cr = (CRootClass *)child.SafeCast(&CRootClass::Type);
    std::cout << cr->name << std::endl;

    return 0;
}

```

# 在持久化数据时的运用
待补充
