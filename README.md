﻿# chatbot-llama-con-bugs-
import streamlit as st
from langchain_community.llms import Ollama 
from langchain_core.messages import HumanMessage, AIMessage, SystemMessage
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder

llm = Ollama(model="llama3.1:latest")

def main():
    st.title("Chat with Llama")
    bot_name = st.text_input("nombre del asistente virtual", value="Bot")
    prompt = (f"sos un asistente virtual llamado {bot_name}. respondes preguntas simples y repreguntas para saber más del usuario")
    bot_descrip = st.text_area("descripcion del asistente virtual", value=prompt)

    if "chat_history" not in st.session_state:
        st.session_state["chat_history"] = []
    
    # Crear mensajes como instancias de SystemMessage y MessagesPlaceholder
    pront_template = ChatPromptTemplate.from_messages(
        [
            SystemMessage(content=bot_descrip),
            MessagesPlaceholder(variable_name="chat_history"),
            HumanMessage(content="{input}"),
        ]
    )

    chain = pront_template | llm
    user_input = st.text_area("escribe aqui tu pregunta", key="user_input")

    if st.button("enviar"):
        if user_input.lower() == "chau":
            st.stop()
        else:
            response = chain.invoke({"input": user_input, "chat_history": st.session_state["chat_history"]})
            st.session_state["chat_history"].append(HumanMessage(content=user_input))
            st.session_state["chat_history"].append(AIMessage(content=response))
    
    chat_display = ""
    for msg in st.session_state["chat_history"]:
        if isinstance(msg, HumanMessage):
            chat_display += f"humano: {msg.content}\n"
        elif isinstance(msg, AIMessage):
            chat_display += f"{bot_name}: {msg.content}\n"
    
    st.text_area("chat", value=chat_display, height=400, key="chat_area", disabled=True)

if __name__ == "__main__":
    main()
